# FreeAssetsHub — /assets/ Folder

## What this folder is for

This folder holds **vault.js** — the encrypted file storage engine for FreeAssetsHub.

All asset files you upload via the Admin Panel are:
1. Encrypted with **AES-256-GCM** (military-grade) using a key derived via PBKDF2
2. Stored in **IndexedDB** (the browser's built-in database — no server needed)
3. Decrypted on-demand only when a user clicks "Download Now" after the ad countdown

## How it works

```
Admin uploads file  →  vault.js encrypts it  →  stores in IndexedDB
                                                        ↓
User clicks Download →  vault.js decrypts it  →  browser Save dialog
```

## Files in this folder

| File | Purpose |
|------|---------|
| `vault.js` | Core AES-256-GCM encrypt/decrypt engine |

## Vault API (used by admin/dashboard.html)

```js
// Store a file (from file input)
await VAULT.store(assetId, file)      // returns { ok, assetId, size }

// Download / decrypt a file
await VAULT.download(assetId, 'filename.png')  // triggers browser save dialog

// Check if a file exists
await VAULT.has(assetId)              // returns true/false

// List all stored files
await VAULT.list()                    // returns [{ id, assetId, meta }]

// Delete one file
await VAULT.remove(assetId)           // returns { ok }

// Export all encrypted data as backup JSON
await VAULT.exportVault()             // downloads a .json backup file

// Import from a backup file
await VAULT.importVault(file)         // re-loads encrypted records
```

## Security Notes

- The encryption key is derived from `VAULT_SECRET` constant in `vault.js`
  — change this string before deploying to production
- Ciphertext stays encrypted at rest in IndexedDB
- Each file gets a unique random IV (Initialization Vector)
- The secret is in client-side JS — this protects against casual snooping
  and disk-level access, but not against someone who inspects the JS source
- For production, move the VAULT_SECRET server-side and use a backend API

## Backup & Restore

1. Admin Panel → Vault Manager → **Export Backup**
   → downloads `FAH_vault_backup_TIMESTAMP.json`
2. On a new browser/machine: Admin Panel → Vault Manager → **Import Backup**
   → re-loads all encrypted records

The export file contains ciphertext only — files cannot be read from the
backup without also having the same `VAULT_SECRET`.
