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
| Database | Persist per‑workspace keys, control access via RLS | `workspace_secrets` table (`20251115120000_secure_patient_files.sql`) |
| Edge Function | Provide a derived key when the table is missing (CI not run, first deploy, etc.) | `supabase/functions/workspace-key` |
| Client Hooks | Fetch/ensure keys, encrypt/decrypt blobs, orchestrate uploads & downloads | `useWorkspaceEncryptionKey`, `usePatientFiles` |
| UI Layer | Consume hooks, display status/errors, trigger uploads/downloads | `PatientFilesSection`, `UniversalViewer`, ZIP download logic |

---

## 3. Implementation Checklist

### 3.1 Database Migration

1. **Create `workspace_secrets` table**
   - Columns: `workspace_id`, `encryption_key`, `rotation_version`, `created_by`, timestamps.
   - Add `update_workspace_secrets_updated_at` trigger.
   - Enable RLS and add policies:
     - Members can read.
     - Owners/Admins can upsert/delete.
2. **Augment target metadata table**
   - For each logical record referencing stored blobs (e.g., `patient_files`), add:
     - `is_encrypted boolean NOT NULL DEFAULT false`
     - `encryption_metadata jsonb`
3. **Update existing rows**
   - Set `is_encrypted` to `false` explicitly to avoid `NULL` semantics.
4. **Lock down the bucket**
   - `UPDATE storage.buckets SET public = false WHERE id = '<bucket>';`
5. **RLS alignment**
   - Ensure `storage.objects` policies already gate by workspace; reuse existing patterns from `patient-files`.

> **Tip:** Always include `DROP POLICY IF EXISTS …` before creating policies in migrations to keep them idempotent.

### 3.2 Edge Function (`workspace-key`)

Purpose: Derive a deterministic fallback key whenever the migration is not yet available.

Key points:
- Validates the requesting user (JWT).
- Checks membership in the requested workspace.
- Derives a base64 key via `SHA-256(workspace_id + WORKSPACE_KEY_MASTER_SECRET)`.
- Returns metadata `{ encryption_key, rotation_version: -1, source: "fallback" }`.
- Deploy with `supabase functions deploy workspace-key`.

Environment:
- `WORKSPACE_KEY_MASTER_SECRET` optional (defaults to `SUPABASE_SERVICE_ROLE_KEY`).

### 3.3 Client Hook (`useWorkspaceEncryptionKey`)

Responsibilities:
1. Fetch secret from `workspace_secrets`.
2. Handle 404/`42P01` gracefully by calling the edge function.
3. Cache secrets in state.
4. Provide:
   - `ensureEncryptionKey()`
   - `rotateEncryptionKey()` (disabled while in fallback mode)
   - `canManageKey` flag and error messaging.

Best practices:
- Detect missing relations by checking error code `42P01`.
- When fallback is active, surface a warning toast so admins know migrations must run.
- Never return the key unless the user is a workspace member.

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
   - Call `ensureEncryptionKey()`.
   - Encrypt the `File` via `encryptBlobWithMetadata`.
   - Store metadata + `is_encrypted = true`.
2. Download/preview:
   - Fetch blob from storage.
   - If `is_encrypted`, decrypt with metadata.
   - Expose helper `loadFileForPreview` so UI components can reuse the logic.
3. Bulk ZIP exports:
   - Iterate files, call `getFileBlob`, zip buffers.
4. Legacy files:
   - If `is_encrypted` is `false`, skip decryption but left shift users to reupload.

### 3.6 UI Integration

- `PatientFilesSection` should use typed `PatientFile`.
- `UniversalViewer` now accepts `resolver` returning decrypted blob/buffer.
- `FileItem`, `DriveItemCard`, etc., call `handleViewFile` without storing secrets locally.
- Display toasts when fallback mode is active (optional but recommended).

---

## 4. Deployment Workflow

1. **Local Prep**
   - Write migration, hook changes, UI updates.
   - Add tests for crypto utilities.
2. **Apply Locally**
   - `supabase db reset` (local) or `supabase start`.
3. **Push to Remote**
   - `supabase db push` (ensure CLI is up to date).
   - If remote has extra migrations, run `supabase migration repair` per CLI hints.
4. **Deploy Functions**
   - `supabase functions deploy workspace-key`.
5. **Verify**
   - `supabase migration list` (local == remote).
   - `supabase db remote exec "SELECT COUNT(*) FROM workspace_secrets;"`.

---

## 5. Best Practices & Lessons Learned

1. **Always seed data with deterministic defaults**
   - Set `is_encrypted` explicitly for legacy rows to avoid null semantics.
2. **Policy collisions**
   - Wrap migrations with `DROP POLICY IF EXISTS` to keep re‑runs idempotent.
3. **Fallback mode is temporary**
   - Surface warnings so admins know to run migrations; disable rotation until the table exists.
4. **Avoid storing secrets client‑side**
   - Retrieve keys on demand; never persist in localStorage or Zustand stores.
5. **Test real downloads**
   - ZIP downloads and `UniversalViewer` should invoke resolvers instead of accessing Supabase directly.
6. **Bucket privacy**
   - Even with encryption, keep buckets non‑public to prevent enumeration.
7. **Error transparency**
   - Log precise errors (404 vs `42P01`) so diagnosing missing migrations is simpler.

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
| 404 on `workspace_secrets` | Migration not applied | Run `supabase db push`, verify table |
| CORS on `workspace-key` | Function unset | Deploy via `supabase functions deploy workspace-key` |
| `The provided data is too small` | Wrong IV/key pair | Ensure metadata persisted; reupload file |
| Legacy file not decrypting | `is_encrypted = false` | Reupload or add migration to encrypt existing blobs |
| Rotation throws | Fallback mode active | Run migrations, ensure table exists, retry |

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

