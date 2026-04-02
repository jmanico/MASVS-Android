# MASVS-STORAGE-ANDROID: Secure Data Storage

Android-specific requirements for MASVS-STORAGE-1 and MASVS-STORAGE-2.

## MASVS-STORAGE-1: Secure Storage of Sensitive Data

### STORAGE-ANDROID-1.1: Hardware-Backed Key Storage

The app MUST generate and store cryptographic keys in Android Keystore with hardware backing (TEE or StrongBox). Keys MUST NOT be imported from external sources when hardware generation is available.

**Testable:** Verify via key attestation that `KeyInfo.isInsideSecureHardware()` returns true.

### STORAGE-ANDROID-1.2: StrongBox Usage for High-Value Keys

For apps processing financial data or sensitive credentials, cryptographic keys SHOULD use StrongBox (physically separate HSM, FIPS 140-2 Level 3) when available on the device.

**Testable:** Verify `setIsStrongBoxBacked(true)` is set in `KeyGenParameterSpec` for high-value key operations. Verify graceful fallback to TEE-backed Keystore when StrongBox is unavailable.

### STORAGE-ANDROID-1.3: EncryptedSharedPreferences for Local Data

The app MUST use `EncryptedSharedPreferences` (Jetpack Security library) for any sensitive key-value data stored locally. Plain `SharedPreferences` MUST NOT store sensitive data including tokens, credentials, PII, or session identifiers.

**Testable:** Static analysis confirms no sensitive data written to `SharedPreferences`. Dynamic analysis confirms encrypted storage files under `shared_prefs/`.

### STORAGE-ANDROID-1.4: Encrypted Database Storage

If the app uses local databases for sensitive data, it MUST use encrypted storage such as SQLCipher with Room or equivalent. The database encryption key MUST be stored in Android Keystore, not hardcoded or derived from predictable values.

**Testable:** Inspect database files on disk for plaintext content. Verify encryption key origin traces to Keystore.

### STORAGE-ANDROID-1.5: External Storage Prohibition

The app MUST NOT write sensitive data to external storage (SD card, public Downloads, or any world-readable location).

**Testable:** Monitor file I/O during app operation. No sensitive data appears in `/sdcard/`, `Environment.getExternalStorageDirectory()`, or `Context.getExternalFilesDir()` paths.

### STORAGE-ANDROID-1.6: Credential Storage Requirements

Access tokens MUST be stored in memory or encrypted storage with a 5-15 minute lifetime. Refresh tokens MUST be stored in Android Keystore. Credentials MUST NOT be stored in `SharedPreferences`, `NSUserDefaults`, `localStorage`, or unencrypted SQLite databases.

**Testable:** Inspect token storage locations. Verify access token expiry configuration. Confirm refresh tokens are Keystore-protected.

## MASVS-STORAGE-2: Prevention of Sensitive Data Leakage

### STORAGE-ANDROID-2.1: Backup Exclusion

The app MUST set `android:allowBackup="false"` or use `android:dataExtractionRules` (Android 12+) to explicitly exclude sensitive data from ADB and cloud backups.

**Testable:** Inspect `AndroidManifest.xml` for backup configuration. Attempt ADB backup and verify no sensitive data is extractable.

### STORAGE-ANDROID-2.2: Log Sanitization

The app MUST NOT log sensitive data (tokens, credentials, PII, cryptographic material) to Android system logs (`Log.d`, `Log.v`, `Log.i`, etc.) in release builds.

**Testable:** Run app with `adb logcat` and exercise all features. No sensitive data appears in log output.

### STORAGE-ANDROID-2.3: Screenshot and Task Switcher Protection

Activities displaying sensitive data MUST set `FLAG_SECURE` on the window to prevent screenshots and task-switcher thumbnail capture.

**Testable:** Attempt screenshot on sensitive screens. Verify task-switcher shows blank/obscured thumbnail.

### STORAGE-ANDROID-2.4: Clipboard Protection

The app SHOULD NOT copy sensitive data to the system clipboard. If clipboard use is necessary for sensitive data, the app MUST use `ClipData.newPlainText()` with a content expiration or instruct users about the risk.

**Testable:** Monitor clipboard contents during app use. Verify no persistent sensitive data in clipboard.

### STORAGE-ANDROID-2.5: Keyboard Cache Prevention

For input fields accepting sensitive data (passwords, credit cards, OTPs), the app MUST disable keyboard suggestions and caching using `android:inputType="textNoSuggestions|textPassword"` or equivalent.

**Testable:** Inspect input field XML attributes. Verify keyboard dictionaries do not cache sensitive input.

### STORAGE-ANDROID-2.6: WebView Cache and Storage Cleanup

If the app uses WebViews, it MUST clear WebView cache, cookies, and local storage when the user logs out or the session ends.

**Testable:** After logout, inspect WebView data directories for residual sensitive data.

### STORAGE-ANDROID-2.7: OTP Protection

On Android 15+, the app SHOULD leverage OTP protection features that hide OTPs from notification previews.

**Testable:** Send OTP to device. Verify notification content does not expose the OTP code on lock screen.
