# MASVS-CODE-1

## Control

The app requires an up-to-date platform version.

## Description

Each Android release includes security patches, privacy improvements, and new security features. Supporting older Android versions keeps users exposed to known, patched vulnerabilities and prevents the app from leveraging modern security mechanisms. This control ensures the app targets and requires a sufficiently recent Android version to benefit from current platform security protections.

## Android Sub-Requirements

### MASVS-CODE-1.1 — Set Appropriate minSdkVersion

The app's `minSdkVersion` is set to at least API 28 (Android 9) to ensure baseline security features including:
- Network Security Configuration with cleartext blocking by default
- StrongBox Keystore support
- BiometricPrompt API availability
- Protected Confirmation support
- Hardware security module attestation

For new apps, `minSdkVersion` of API 29 (Android 10) or higher is recommended to benefit from scoped storage, background location restrictions, and TLS 1.3 by default.

**Android References:**
- `minSdkVersion` in `build.gradle` / `build.gradle.kts`

### MASVS-CODE-1.2 — Target the Latest Stable API Level

The app's `targetSdkVersion` is set to the latest stable API level (API 35 for Android 15, API 36 for Android 16) to opt in to the latest security behaviors:
- API 31+: `PendingIntent` mutability must be explicit; components with intent-filters must declare `android:exported`
- API 33+: `RECEIVER_NOT_EXPORTED` flag for broadcast receiver registration
- API 34+: Partial media access; stricter background restrictions
- API 35+: TLS 1.0/1.1 disabled; safer intent matching
- API 36+: Local network permission; Safer Intents v2; intent redirection hardening

**Rationale:** Google Play requires targeting a recent API level. Each new target SDK activates security behavior changes that protect the app and its users.

### MASVS-CODE-1.3 — Prompt Users on Outdated OS Versions

If the app detects it is running on an Android version that no longer receives security patches (typically versions more than 3 years old), it informs the user of the security risk. The app may choose to restrict sensitive functionality on unsupported OS versions.

**Rationale:** Even with a current app, an outdated OS with unpatched kernel and framework vulnerabilities puts user data at risk.

### MASVS-CODE-1.4 — Use Android API Level Checks for Security Features

The app uses `Build.VERSION.SDK_INT` checks to leverage stronger security features on newer API levels while maintaining compatibility:
```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
    // Use StrongBox
} else {
    // Fall back to TEE
}
```

**Rationale:** Progressive security enhancement ensures users on newer devices get the strongest available protections while older devices still function.
