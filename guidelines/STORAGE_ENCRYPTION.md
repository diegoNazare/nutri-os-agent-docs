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
| Database | Persist salted KDF metadata + fingerprints (no plaintext keys) | `workspace_key_metadata` table (`20251117120000_e2ee_workspace_keys.sql`) |
| Edge Functions | *Nenhum* – removidos para garantir conhecimento zero no servidor | n/a |
| Client Hooks | Derivam/retêm chaves apenas em memória, conduzem AES-GCM local | `useWorkspaceEncryptionKey`, `usePatientFiles` |
| UI Layer | Solicita passphrase, bloqueia/desbloqueia acesso, aciona fluxos | `PatientFilesSection`, `DocumentViewer`, ZIP download logic |

---

## 3. Implementation Checklist

### Workspace Passphrase & KDF

- Cada workspace define uma passphrase de alta entropia, compartilhada somente entre profissionais autorizados (fora do sistema).
- A chave simétrica é derivada exclusivamente no cliente via PBKDF2 (salt aleatório + parâmetros armazenados no banco).
- O servidor guarda apenas `kdf_salt`, `kdf_params` e `key_fingerprint` (SHA-256 da chave), impossibilitando reconstruir o segredo.
- Chaves nunca são persistidas em localStorage nem enviadas ao backend; vivem apenas em memória durante a sessão.

### 3.1 Database Migration

1. **Criar `workspace_key_metadata`**
   - Colunas: `workspace_id`, `kdf_salt`, `kdf_params`, `key_fingerprint`, `created_by`, timestamps.
   - Trigger `update_workspace_key_metadata_updated_at`.
   - RLS:
     - Seleção liberada para todos os membros do workspace (precisam do salt).
     - Escrita restrita a owners/admins.
2. **Atualizar `patient_files`**
   - Adicionar `encryption_version integer NOT NULL DEFAULT 2`.
   - Setar `encryption_version = 1` em arquivos legados (`is_encrypted = true`).
3. **Remover heranças**
   - Dropar `workspace_secrets`, gatilhos e policies antigas.
4. **Bucket privado**
   - `UPDATE storage.buckets SET public = false WHERE id = '<bucket>';`
5. **RLS alignment**
   - `storage.objects` já utiliza vínculos de workspace; revise apenas se novos buckets forem protegidos.

> **Tip:** Always include `DROP POLICY IF EXISTS …` before creating policies in migrations to keep them idempotent.

### 3.2 Edge Function (`workspace-key`)

Purpose: Implementar derivação e validação 100% no cliente, sem qualquer dependência de segredos no servidor.

Fluxo:
- Owners configuram uma passphrase forte e, ao inicializar, o cliente gera `kdf_salt` randômico + `key_fingerprint`.
- O servidor guarda somente `kdf_salt`, `kdf_params` e `key_fingerprint`. Sem a passphrase, não há como recuperar a chave.
- Cada login exige que o usuário digite a passphrase; o hook deriva a chave e mantém em memória (Zustand). Nada é persistido em localStorage.
- Rotação = novo salt + novo fingerprint; membros devem receber a nova passphrase por canal seguro.
- Arquivos com `encryption_version = 1` são considerados legados e precisam ser reenviados manualmente (o servidor não possui, nem deve possuir, uma chave antiga).

### 3.3 Client Hook (`useWorkspaceEncryptionKey`)

Responsabilidades:
1. Carregar `workspace_key_metadata` e refletir estados (`loading`, `uninitialized`, `locked`, `ready`).
2. Derivar a chave usando PBKDF2 e comparar com `key_fingerprint`.
3. Expor helpers:
   - `unlockWorkspace(passphrase)`
   - `initializeWorkspaceKey(passphrase)` / `rotateWorkspaceKey(passphrase)` (apenas owners/admins)
   - `lockWorkspace()` e `requireWorkspaceKey()`
4. Nunca armazenar a chave fora de memória; destruir referências ao trocar de workspace ou ao bloquear manualmente.

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
   - Chame `requireWorkspaceKey()` para garantir que o workspace está desbloqueado.
   - Criptografe o `File` via `encryptBlobWithMetadata`.
   - Armazene metadata + `encryption_version = 2`.
2. Download/preview:
   - Fetch blob from storage.
   - If `is_encrypted`, decrypt with metadata.
   - Expose helper `loadFileForPreview` so UI components can reuse the logic.
3. Bulk ZIP exports:
   - Iterate files, call `getFileBlob`, zip buffers.
4. Legacy files:
   - Se `encryption_version < 2`, bloqueie a visualização e peça reupload.

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
   - `supabase db push` (ensure CLI is up to date) – applies schema + wrapped columns.
   - Run the backfill: `WORKSPACE_KEY_MASTER_SECRET=... SUPABASE_SERVICE_ROLE_KEY=... npx ts-node scripts/backfillWorkspaceSecrets.ts`
   - `supabase db push` again to execute the guard migration that drops `encryption_key`.
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

