# MASVS-RESILIENCE-1

## Control

The app validates the integrity of the platform.

## Description

Running on a compromised Android platform — rooted devices, unlocked bootloaders, custom ROMs, or devices with Magisk/Xposed — undermines many security controls that depend on platform guarantees (sandboxing, secure storage, biometric integrity). This control ensures the app detects platform compromise and responds appropriately, using multiple layered detection techniques and server-side verification via the Google Play Integrity API.

## Android Sub-Requirements

### MASVS-RESILIENCE-1.1 — Use Google Play Integrity API for Device Attestation

The app requests a Play Integrity token on launch or before sensitive operations and sends it to the server for verification. The server evaluates:

| Verdict | Label | Action |
|---|---|---|
| `deviceRecognitionVerdict` | `MEETS_DEVICE_INTEGRITY` | Allow full functionality |
| `deviceRecognitionVerdict` | `MEETS_BASIC_INTEGRITY` only | Warn user; restrict sensitive features |
| `deviceRecognitionVerdict` | No labels | Block sensitive operations |
| `appRecognitionVerdict` | `PLAY_RECOGNIZED` | App is genuine |
| `appRecognitionVerdict` | Not recognized | App may be modified; restrict |
| `appLicensingVerdict` | `LICENSED` | App acquired from Play Store |

**Critical:** All verdict evaluation must happen server-side. The client only obtains and forwards the integrity token; it never evaluates verdicts locally.

**Android References:**
- `com.google.android.play:integrity` Gradle dependency
- `IntegrityManager.requestIntegrityToken(IntegrityTokenRequest)` — client-side
- Google Play servers for decryption/verification — server-side

### MASVS-RESILIENCE-1.2 — Implement Multi-Signal Root Detection

In addition to Play Integrity, the app performs local root detection checks as defense-in-depth:

| Signal | Detection Method |
|---|---|
| `su` binary | Check `/system/bin/su`, `/system/xbin/su`, `/sbin/su`, `/system/app/Superuser.apk` |
| Root management apps | Check for Magisk Manager, SuperSU, KingRoot packages via `PackageManager` |
| Build properties | Check `ro.build.tags` for "test-keys", `ro.debuggable` for "1" |
| SELinux status | Verify SELinux is in "enforcing" mode |
| System partition | Check `/system` is mounted read-only |
| Mount points | Parse `/proc/mounts` for Magisk overlay mounts |
| Xposed/LSPosed | Check for Xposed Installer package; check `/proc/self/maps` for Xposed libraries |

**Important:** Local root detection is a speed bump, not a security boundary. Determined attackers can bypass these checks. Play Integrity with hardware-backed verdicts (Android 13+) is significantly harder to bypass.

### MASVS-RESILIENCE-1.3 — Perform Root Checks in Native Code

Critical root detection logic is implemented in native code (C/C++ via JNI) in addition to Java/Kotlin, cross-referencing results from both layers.

**Rationale:** Java/Kotlin method calls are trivially hookable with Frida or Xposed. Native code requires more sophisticated instrumentation to bypass. Cross-referencing Java and native results detects single-layer hooks.

### MASVS-RESILIENCE-1.4 — Detect Emulators

The app detects execution on emulators by checking:
- `Build.FINGERPRINT` contains "generic", "sdk", "google_sdk"
- `Build.MODEL` contains "Emulator", "Android SDK", "sdk_gphone"
- Emulator-specific files exist (`/dev/qemu_pipe`, `/dev/goldfish_pipe`)
- `TelephonyManager` returns known-fake IMEI/IMSI values
- Hardware sensors return unrealistic or absent readings

**Rationale:** Emulators allow easy app instrumentation, debugging, and traffic interception. Detecting emulators prevents casual reverse engineering and automated scraping.

### MASVS-RESILIENCE-1.5 — Respond Appropriately to Platform Integrity Failures

When platform integrity checks fail, the app does not simply display an error and exit (which makes bypass detection trivial). Instead:
- High-risk apps (financial, healthcare): restrict sensitive functionality server-side based on Play Integrity verdicts; continue non-sensitive functionality
- Medium-risk apps: warn the user and log the event; allow usage with reduced trust level
- All apps: never reveal which specific check failed (makes bypass harder)

**Rationale:** Binary "pass/fail" responses are easy to bypass by hooking the single decision point. Graduated, server-side responses based on multiple signals are more resilient.
