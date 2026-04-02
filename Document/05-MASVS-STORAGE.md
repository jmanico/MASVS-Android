# MASVS-STORAGE: Secure Data Storage at Rest

## Overview

Mobile apps frequently store data locally on the device. Android provides multiple storage mechanisms — internal storage (app-sandboxed), external/shared storage, SharedPreferences, SQLite databases, the AndroidKeyStore, and file-based encryption. Each has different security properties. This category ensures that sensitive data is stored using the most appropriate mechanism with proper encryption and access controls.

## Android Storage Landscape

| Storage Location | Security Level | Access | Notes |
|---|---|---|---|
| Internal storage (`getFilesDir()`) | High | App-only (UID sandbox) | Best for sensitive data; encrypted at rest on FBE devices |
| No-backup files (`getNoBackupFilesDir()`) | High | App-only, excluded from backup | For sensitive data that should not be backed up |
| AndroidKeyStore | Highest | App-only, hardware-backed | For cryptographic keys; non-extractable |
| SharedPreferences | Medium | App-only (plaintext XML) | Never for secrets without encryption |
| Room/SQLite database | Medium | App-only (plaintext .db) | Encrypt with SQLCipher for sensitive data |
| External app storage (`getExternalFilesDir()`) | Medium | App-only on API 29+ | Removed on uninstall; was shared on older APIs |
| Shared external storage | Low | Any app with permission | Never for sensitive data |
| ContentProvider | Varies | Depends on export/permissions | Must be properly secured |

## Key Android APIs and Libraries

- **Google Tink** (`com.google.crypto.tink:tink-android`) — Modern cryptographic library with AEAD, streaming AEAD, and deterministic AEAD. Recommended replacement for Jetpack Security.
- **Jetpack DataStore** (`androidx.datastore:datastore-tink`) — Encrypted key-value and proto storage with Tink integration.
- **AndroidKeyStore** — Hardware-backed key storage (TEE/StrongBox). Keys never enter app process memory.
- **SQLCipher** (`net.zetetic:android-database-sqlcipher`) — Transparent SQLite encryption compatible with Room.
- **EncryptedSharedPreferences** — **Deprecated** (April 2025). Migrate to DataStore + Tink.

## Controls

| Control | Sub-Requirements | Focus |
|---|---|---|
| [MASVS-STORAGE-1](../controls/MASVS-STORAGE-1.md) | 9 sub-requirements | Securely storing sensitive data |
| [MASVS-STORAGE-2](../controls/MASVS-STORAGE-2.md) | 9 sub-requirements | Preventing unintentional data leakage |

## OWASP Mobile Top 10 2024 Mapping

- **M9 — Insecure Data Storage:** Directly addressed by MASVS-STORAGE-1
- **M1 — Improper Credential Usage:** Addressed by MASVS-STORAGE-1.3, 1.4, 1.6
- **M8 — Security Misconfiguration:** Addressed by MASVS-STORAGE-1.5, 1.8, MASVS-STORAGE-2.1
