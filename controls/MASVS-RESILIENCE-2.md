# MASVS-RESILIENCE-2

## Control

The app implements anti-tampering mechanisms.

## Description

Without protection, it is relatively straightforward to modify an Android APK — re-signing with a different key after code or resource modification, patching DEX bytecode, or uploading a backdoored version to third-party stores. This control ensures the app detects modifications to its own code, resources, and signing certificate to maintain integrity.

## Android Sub-Requirements

### MASVS-RESILIENCE-2.1 — Verify App Signing Certificate at Runtime

The app verifies its own signing certificate at runtime against a hardcoded expected certificate hash. This detects repackaging with a different signing key.

**Android References:**
```kotlin
val packageInfo = packageManager.getPackageInfo(packageName,
    PackageManager.GET_SIGNING_CERTIFICATES)
val signatures = packageInfo.signingInfo.apkContentsSigners
val certHash = MessageDigest.getInstance("SHA-256").digest(signatures[0].toByteArray())
// Compare certHash against expected value
```

**Rationale:** Repackaged APKs must be signed with a different key (the attacker does not have the original signing key). Certificate verification detects this.

### MASVS-RESILIENCE-2.2 — Verify APK Integrity via Play Integrity

Use the Play Integrity API's `appRecognitionVerdict` to verify server-side that the APK matches the version published on Google Play (`PLAY_RECOGNIZED`). This detects modified APKs, sideloaded versions, and third-party store distributions.

**Rationale:** Client-side integrity checks can be patched out. Server-side verification via Play Integrity is authoritative and resistant to client-side tampering.

### MASVS-RESILIENCE-2.3 — Detect DEX and Native Library Modifications

The app computes checksums of its DEX files (`classes.dex`, `classes2.dex`, etc.) and native libraries (`.so` files) at runtime and compares them against expected values embedded in the app (or verified server-side).

**Rationale:** Attackers modify DEX bytecode to bypass security checks, inject malicious code, or enable premium features. Checksum verification detects these modifications.

### MASVS-RESILIENCE-2.4 — Use v2+ APK Signing Scheme

The app is signed with APK Signature Scheme v2 or v3 (not v1 JAR signing alone). v2+ signing covers the entire APK file, preventing modification of any content (including resources, assets, and manifest) without invalidating the signature.

**Android References:**
- Use `apksigner` (not `jarsigner`) for v2/v3 signing
- v3 signing (API 28+) enables key rotation via proof-of-rotation
- v3.1 (API 33+) for rotation-only targeting on Android 13+

### MASVS-RESILIENCE-2.5 — Detect Installation Source

The app verifies its installation source using `PackageManager.getInstallSourceInfo()` (API 30+) or `PackageManager.getInstallerPackageName()`. Apps distributed via Google Play should verify the installer is `com.android.vending`.

**Rationale:** Apps sideloaded from third-party sources or installed via ADB may have been modified. Installation source verification is an additional signal (not a sole defense) for detecting unofficial distributions.

### MASVS-RESILIENCE-2.6 — Implement Redundant Integrity Checks

Integrity checks are implemented in multiple locations throughout the app's execution flow (not just on launch) and use different techniques (certificate check, DEX checksum, Play Integrity, native verification). No single point of failure allows bypassing all checks.

**Rationale:** A single integrity check at startup is trivially bypassed by patching one method. Distributed, diverse checks require the attacker to find and bypass all of them, significantly increasing the effort.
