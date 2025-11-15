# Storage Encryption Playbook

This guide documents how to add end‑to‑end encryption to any Supabase storage bucket in Nutri‑OS. It captures the steps we used for the `patient-files` bucket and generalizes them so future buckets can reuse the same approach.

---

## 1. Objectives

1. Ensure that blobs stored in Supabase are unreadable by Supabase admins or anyone outside the workspace context.
2. Keep encryption keys workspace‑scoped so revoking access to one workspace never exposes data from another.
3. Maintain a deterministic process that works even before new database artifacts are deployed (fallback mode).

---

## 2. Architecture Overview

| Layer | Responsibility | Current Implementation |
| --- | --- | --- |
| Database | Store salted KDF metadata + fingerprints (never plaintext keys) | `workspace_key_metadata` (`20251117120000_e2ee_workspace_keys.sql`) |
| Edge Functions | *None* – removed to guarantee zero-knowledge server behavior | n/a |
| Client Hooks | Derive/retain keys only in memory, run AES-GCM locally | `useWorkspaceEncryptionKey`, `usePatientFiles` |
| UI Layer | Prompt passphrase, manage locked/unlocked state, trigger flows | `PatientFilesSection`, `DocumentViewer`, ZIP download logic |

---

## 3. Implementation Checklist

### Workspace Passphrase & KDF

- Each workspace chooses a high-entropy passphrase and shares it only with authorized clinicians through an out-of-band channel.
- The symmetric key is derived exclusively in the browser via PBKDF2 (random salt + stored parameters).
- The server stores only `kdf_salt`, `kdf_params`, and `key_fingerprint` (SHA-256 of the key), making reconstruction impossible.
- Keys are never persisted in localStorage nor sent to the backend; they live only in memory for the current session.

### 3.1 Database Migration

1. **Create `workspace_key_metadata`**
   - Columns: `workspace_id`, `kdf_salt`, `kdf_params`, `key_fingerprint`, `created_by`, timestamps.
   - Trigger: `update_workspace_key_metadata_updated_at`.
   - RLS:
     - Allow `SELECT` for every workspace member (they need the salt/params).
     - Restrict inserts/updates to owners and admins.
2. **Update `patient_files`**
   - Add `encryption_version integer NOT NULL DEFAULT 2`.
   - Backfill `encryption_version = 1` for legacy encrypted rows (`is_encrypted = true`).
3. **Remove legacy artifacts**
   - Drop `workspace_secrets`, related triggers, and policies.
4. **Keep buckets private**
   - `UPDATE storage.buckets SET public = false WHERE id = '<bucket>';`
5. **RLS alignment**
   - Ensure `storage.objects` policies reference the correct workspace relationships when onboarding new buckets.

> **Tip:** Always include `DROP POLICY IF EXISTS …` before creating policies in migrations to keep them idempotent.

### 3.2 Client-Side Passphrase Flow

Purpose: enforce 100% client-run derivation/validation with zero server knowledge.

Flow:
- Owners configure a strong passphrase and, during initialization, the client generates a random `kdf_salt` plus `key_fingerprint`.
- The server stores only `kdf_salt`, `kdf_params`, and `key_fingerprint`. Without the passphrase, the key cannot be recovered.
- Every login requires re-entering the passphrase; the hook derives the key and keeps it in a volatile Zustand store (never persisted).
- Rotation = new salt + new fingerprint; members must receive the new passphrase via a secure out-of-band channel.
- Files with `encryption_version = 1` are legacy and must be re-uploaded manually (no server-side key exists anymore).

### 3.3 Client Hook (`useWorkspaceEncryptionKey`)

Responsibilities:
1. Load `workspace_key_metadata` and expose status flags (`loading`, `uninitialized`, `locked`, `ready`).
2. Derive the key with PBKDF2 and validate it against `key_fingerprint`.
3. Provide helpers:
   - `unlockWorkspace(passphrase)`
   - `initializeWorkspaceKey(passphrase)` / `rotateWorkspaceKey(passphrase)` (owners/admins only)
   - `lockWorkspace()` and `requireWorkspaceKey()`
4. Never store the key anywhere permanent; wipe it when switching workspaces or when the user locks the session.

### 3.4 Encryption Utilities (`src/lib/security/crypto.ts`)

Expose helpers:
- `encryptBlobWithMetadata(blob, key)`
- `decryptBlobFromMetadata(blob, key, metadata)`
- Support browsers lacking `Blob.arrayBuffer()` by using `Response`.
- Export metadata interface (`algorithm`, `iv`, `version`, `original_mime_type`, sizes).

Add unit tests (`crypto.test.ts`) covering:
- Key generation.
- Buffer encryption/decryption.
- Metadata round‑trip.

### 3.5 Domain Hook (`usePatientFiles` Example)

1. Before uploading:
   - Call `requireWorkspaceKey()` to ensure the workspace is unlocked.
   - Encrypt the `File` via `encryptBlobWithMetadata`.
   - Persist metadata plus `encryption_version = 2`.
2. Download/preview:
   - Fetch the raw blob from storage.
   - If `is_encrypted`, decrypt with the saved metadata.
   - Provide a reusable helper (`loadFileForPreview`) so UI components share the same logic.
3. Bulk ZIP exports:
   - Iterate through files, call `getFileBlob`, and zip the decrypted buffers.
4. Legacy files:
   - If `encryption_version < 2`, block inline viewing and request a re-upload.

### 3.6 UI Integration

- `PatientFilesSection` relies on the workspace key status to show setup/unlock forms.
- `UniversalViewer` expects a resolver that returns { blob, buffer } already decrypted.
- `FileItem`, `DriveItemCard`, etc., call `handleViewFile` without persisting any secrets.
- Surface friendly errors when users try to act while the workspace is locked.

---

## 4. Deployment Workflow

1. **Local prep**
   - Author migrations, hooks, and UI changes.
   - Add/refresh crypto unit tests.
2. **Run locally**
   - `supabase db reset` (or `supabase start`) to validate the migrations end-to-end.
3. **Push to remote**
   - `supabase db push` (keep the CLI current).
   - If legacy rows remain, run the cleanup/backfill script before re-running `supabase db push`.
   - Use `supabase migration repair` whenever histories diverge per CLI hints.
4. **Deploy frontend/backend**
   - Redeploy the frontend so the new client-only hooks reach users.
   - No edge function deployment is required anymore.
5. **Verify**
   - `supabase migration list` should show parity between local and remote.
   - Confirm `workspace_secrets` is empty or dropped; `workspace_key_metadata` should hold only salts/fingerprints.

---

## 5. Best Practices & Lessons Learned

1. **Seed deterministic defaults**
   - Explicitly set `is_encrypted`/`encryption_version` for legacy rows to avoid NULL ambiguity.
2. **Handle policy churn gracefully**
   - Prepend `DROP POLICY IF EXISTS …` so repeated migrations stay idempotent.
3. **Fallback mode is temporary**
   - Warn admins when metadata is missing so they know to run migrations; disallow rotation until metadata exists.
4. **Never persist secrets client-side**
   - Fetch keys on demand and keep them only in memory; avoid localStorage/sessionStorage entirely.
5. **Test real downloads**
   - ZIP flows and `UniversalViewer` must use the resolver pattern, never raw signed URLs.
6. **Keep buckets private**
   - Encryption doesn’t replace RLS; private buckets prevent brute-force enumeration.
7. **Log precise errors**
   - Differentiate between missing migrations (e.g., `42P01`) and auth failures to speed up debugging.

---

## 6. Applying to New Buckets

1. Duplicate the migration template:
   - Create `<timestamp>_secure_<bucket>.sql`.
   - Replace table references (e.g., `patient_files` → `<entity>`).
2. Reuse `useWorkspaceEncryptionKey`.
3. Create a domain‑specific hook mirroring `usePatientFiles`.
4. Update UI components to consume the new hook.
5. Document the integration in this playbook (append new section for each bucket).

---

## 7. Troubleshooting

| Symptom | Likely Cause | Fix |
| --- | --- | --- |
| 404 on `workspace_key_metadata` | Migration not applied | Run `supabase db push`, confirm table exists |
| Files fail to decrypt | Workspace locked | Prompt passphrase; ensure `useWorkspaceEncryptionKey` is ready |
| “Data too small” errors | Mismatched IV/key pair | Verify metadata stored correctly, request re-upload |
| Rotation fails | Metadata missing or stale | Re-run migrations, recreate metadata, then rotate |
| Users stuck in fallback state | Old code or missing migrations | Deploy latest frontend and rerun DB migrations |

---

## 8. References

- `.agents/guidelines/ARCHITECTURE.md` – Hook patterns and layering.
- `.agents/guidelines/LOCAL_DEVELOPMENT.md` – Running Supabase locally.
- `.agents/guidelines/PRODUCTION.md` – Deployment expectations.
- `supabase/functions/workspace-key` – Edge function source of truth.
- `src/hooks/useWorkspaceEncryption.ts` – Current hook implementation.
- `src/hooks/usePatientFiles.ts` – Concrete encryption usage example.

---

Keep this file updated as we encrypt more buckets. Document any deviations, schema changes, or new helper utilities so future contributors have a single reference.ঞ

