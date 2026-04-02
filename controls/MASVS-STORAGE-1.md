# MASVS-STORAGE-1

## Control

The app securely stores sensitive data.

## Description

Android apps handle sensitive data from many sources — user input, backend APIs, system services, and other apps — and frequently need to persist it locally. Android provides multiple storage locations with different security properties: internal storage (app-sandboxed), external storage (shared), SharedPreferences, databases, and the AndroidKeyStore. This control ensures that any sensitive data intentionally stored by the app is properly protected using Android-specific storage mechanisms, encryption APIs, and access controls.

## Android Sub-Requirements

### MASVS-STORAGE-1.1 — Use Internal Storage for Sensitive Data

The app stores all sensitive data (credentials, tokens, PII, financial data) exclusively in internal storage (`Context.getFilesDir()`, `Context.getCacheDir()`), never on external/shared storage.

**Rationale:** Internal storage is sandboxed per-app by the Linux kernel (UID separation) and is not accessible to other apps without root. External storage (`Environment.getExternalStorageDirectory()`) is accessible to any app with `READ_EXTERNAL_STORAGE` or via USB/ADB.

**Android References:**
- `Context.getFilesDir()` — `/data/data/<package>/files/`
- `Context.getCacheDir()` — `/data/data/<package>/cache/`
- `Context.getNoBackupFilesDir()` — excluded from Auto Backup

### MASVS-STORAGE-1.2 — Encrypt Sensitive Data at Rest

All sensitive data stored locally is encrypted using authenticated encryption. The app uses `Google Tink` with `AndroidKeysetManager` backed by the AndroidKeyStore, or `Jetpack DataStore` with `AeadSerializer`.

**Rationale:** If a device is rooted, lost, or backed up, plaintext data in internal storage is exposed. Encryption with hardware-backed keys ensures data remains confidential even if the filesystem is compromised.

**Android References:**
- `com.google.crypto.tink:tink-android` — Tink AEAD (AES-256-GCM)
- `AndroidKeysetManager` — key management backed by AndroidKeyStore
- `androidx.datastore:datastore-tink` — encrypted DataStore with `AeadSerializer`
- **Deprecated:** `EncryptedSharedPreferences` (deprecated April 2025, `security-crypto:1.1.0-alpha07`)

### MASVS-STORAGE-1.3 — Never Store Secrets in SharedPreferences Without Encryption

The app does not store sensitive values (tokens, passwords, API keys, PII) in `SharedPreferences` without encryption. Plain `SharedPreferences` writes XML files to internal storage in cleartext.

**Rationale:** While SharedPreferences files are sandboxed, they are trivially readable on rooted devices, in backups, and via ADB on debuggable builds. Any secret stored in plaintext SharedPreferences should be considered compromised on a rooted device.

### MASVS-STORAGE-1.4 — Encrypt Databases Containing Sensitive Data

Databases containing sensitive data use `SQLCipher` for Android or equivalent transparent database encryption. If using Room, the app provides an `SupportSQLiteOpenHelper.Factory` backed by SQLCipher.

**Rationale:** SQLite databases in internal storage are plaintext `.db` files. On rooted devices or via backup extraction, their contents are trivially readable with any SQLite client.

**Android References:**
- `net.zetetic:android-database-sqlcipher` — SQLCipher for Android
- `androidx.room:room-runtime` with `SupportSQLiteOpenHelper.Factory` override

### MASVS-STORAGE-1.5 — Use MODE_PRIVATE for All File Creation

All files created by the app use `Context.MODE_PRIVATE` (the default since API 17). The app never uses `MODE_WORLD_READABLE` or `MODE_WORLD_WRITABLE` (deprecated since API 17, removed in API 24).

**Rationale:** World-readable/writable files break Android's sandbox model and expose data to any app on the device.

### MASVS-STORAGE-1.6 — Protect AndroidKeyStore Keys with User Authentication

Cryptographic keys protecting sensitive stored data are bound to user authentication via `KeyGenParameterSpec.Builder.setUserAuthenticationRequired(true)` with a defined timeout and authenticator type.

**Rationale:** Without user authentication binding, any process that gains access to the app's UID can use the key. Binding to biometric or device credential ensures the user must be physically present.

**Android References:**
- `KeyGenParameterSpec.Builder.setUserAuthenticationRequired(true)`
- `KeyGenParameterSpec.Builder.setUserAuthenticationParameters(timeout, AUTH_BIOMETRIC_STRONG)`

### MASVS-STORAGE-1.7 — Prefer StrongBox-Backed Key Storage

When storing cryptographic keys for sensitive data, the app requests StrongBox backing via `KeyGenParameterSpec.Builder.setIsStrongBoxBacked(true)` and gracefully falls back to TEE when StrongBox is unavailable.

**Rationale:** StrongBox provides a dedicated secure processor with its own CPU, memory, and storage, offering stronger tamper resistance than the TEE. It is available on devices with `PackageManager.FEATURE_STRONGBOX_KEYSTORE`.

### MASVS-STORAGE-1.8 — Exclude Sensitive Data from Auto Backup

The app either disables Auto Backup (`android:allowBackup="false"`) or configures `android:fullBackupContent` / `android:dataExtractionRules` (API 31+) to explicitly exclude files and directories containing sensitive data.

**Rationale:** Android Auto Backup copies app data to Google Drive. If sensitive data is included, it may be accessible via the user's Google account on other devices or by anyone who compromises the account.

**Android References:**
- `android:allowBackup="false"` in `AndroidManifest.xml`
- `android:dataExtractionRules="@xml/data_extraction_rules"` (API 31+)
- `android:fullBackupContent="@xml/backup_rules"` (API 23-30)
- `Context.getNoBackupFilesDir()` — directory excluded from backup by default

### MASVS-STORAGE-1.9 — Clear Sensitive Data from Memory When No Longer Needed

The app clears sensitive data (passwords, keys, tokens) from memory by overwriting byte arrays and char arrays after use. The app avoids storing secrets in immutable `String` objects where possible.

**Rationale:** Java/Kotlin `String` objects are immutable and remain in the heap until garbage collected. On rooted devices, heap dumps can extract these values. `byte[]` and `char[]` can be explicitly zeroed.
