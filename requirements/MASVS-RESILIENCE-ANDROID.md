# MASVS-RESILIENCE-ANDROID: Resilience Against Reverse Engineering

Android-specific requirements for MASVS-RESILIENCE-1 through MASVS-RESILIENCE-4.

## MASVS-RESILIENCE-1: Reverse Engineering Resistance

### RESILIENCE-ANDROID-1.1: Code Obfuscation with R8

Release builds MUST enable R8 (or ProGuard) with code shrinking, obfuscation, and optimization. Obfuscation MUST rename classes, methods, and fields to hinder static analysis with tools like jadx, apktool, and Ghidra.

**Testable:** Decompile release APK with jadx. Verify class/method names are obfuscated. Verify `minifyEnabled true` in `build.gradle`.

### RESILIENCE-ANDROID-1.2: Native Code Protection

If the app uses native code (JNI/NDK), native libraries SHOULD be stripped of debug symbols and SHOULD use additional obfuscation. Sensitive logic in native code SHOULD use control-flow obfuscation.

**Testable:** Run `nm` or `readelf` on `.so` files. Verify debug symbols are stripped. Verify string obfuscation on sensitive strings.

### RESILIENCE-ANDROID-1.3: String Encryption

Sensitive strings (API endpoints, cryptographic constants, business logic identifiers) SHOULD be encrypted or obfuscated in the APK. They MUST NOT appear as plaintext in the DEX or resource files.

**Testable:** Search decompiled APK for known sensitive strings. Verify they are not present in plaintext.

### RESILIENCE-ANDROID-1.4: Server-Side Sensitive Logic

Sensitive business logic (pricing, entitlement checks, fraud detection) MUST be implemented server-side. Client-side protections are speed bumps, not walls — any client-side check can be bypassed given sufficient effort.

**Testable:** Review app architecture. Verify sensitive decisions are made server-side with client receiving only results.

## MASVS-RESILIENCE-2: Tamper Detection and Response

### RESILIENCE-ANDROID-2.1: Play Integrity API

The app MUST use the Play Integrity API to verify:
- **App integrity:** The app binary is unmodified and from a recognized source
- **Device integrity:** The device meets `MEETS_DEVICE_INTEGRITY` or `MEETS_STRONG_INTEGRITY` tier
- **Account licensing:** The user has a valid Play Store license

Attestation results MUST be verified server-side. Client-side-only verification is insufficient.

**Testable:** Verify Play Integrity API integration. Verify attestation token is sent to server. Verify server validates the token and acts on integrity failures.

### RESILIENCE-ANDROID-2.2: Root/Magisk Detection

The app SHOULD detect rooted devices via multiple signals:
- Presence of `su` binary
- Magisk/Zygisk artifacts
- Modified system partitions
- Play Integrity `MEETS_STRONG_INTEGRITY` tier (fails on rooted devices)

Root detection MUST be verified server-side. Client-side detection alone is always bypassable.

**Testable:** Run app on rooted device with Magisk. Verify detection triggers. Verify server-side enforcement.

### RESILIENCE-ANDROID-2.3: Frida Detection

For high-security apps, the app SHOULD detect common dynamic instrumentation tools (Frida, Xposed) via:
- Process name inspection (frida-server)
- Named pipe detection
- Code section hash verification
- Frida gadget detection

**Testable:** Attach Frida to the app process. Verify detection and appropriate response (e.g., session termination).

### RESILIENCE-ANDROID-2.4: Tamper Response

When tampering or integrity violations are detected, the app MUST:
- Report the violation to the server
- The server MUST decide the enforcement action (block, limit, monitor)
- The app SHOULD NOT silently continue operating in a compromised state

**Testable:** Trigger integrity violation. Verify server receives report. Verify enforcement action is applied.

## MASVS-RESILIENCE-3: File Integrity

### RESILIENCE-ANDROID-3.1: APK Integrity Verification

The app SHOULD verify its own APK signature at runtime to detect repackaging attacks. Verification SHOULD use the `PackageManager` API to check the signing certificate.

**Testable:** Repackage and resign APK with different key. Verify app detects the signature change.

### RESILIENCE-ANDROID-3.2: Resource Integrity

Critical app resources (configuration files, ML models, business rule files) SHOULD have integrity checks (hash verification) to detect modification.

**Testable:** Modify a critical resource file. Verify app detects the modification.

## MASVS-RESILIENCE-4: Client-Side Security Controls

### RESILIENCE-ANDROID-4.1: RASP Integration

High-security apps SHOULD integrate Runtime Application Self-Protection (RASP) solutions for continuous runtime threat detection covering root detection, debugging detection, hooking detection, and emulator detection.

**Testable:** Verify RASP SDK integration. Test detection capabilities against each threat vector.

### RESILIENCE-ANDROID-4.2: Debugger Detection

The app SHOULD detect debugger attachment and respond appropriately. Check for `Debug.isDebuggerConnected()` and `ptrace` status.

**Testable:** Attach a debugger (JDWP or native). Verify detection and response.

### RESILIENCE-ANDROID-4.3: Emulator Detection

The app SHOULD detect execution on emulators for fraud prevention. Check for emulator-specific build properties, sensor availability, and hardware characteristics.

**Testable:** Run app on emulator. Verify detection triggers and appropriate server-side reporting.
