# MASVS-CODE-ANDROID: Code Quality and Security

Android-specific requirements for MASVS-CODE-1 through MASVS-CODE-4.

## MASVS-CODE-1: Secure Build Infrastructure

### CODE-ANDROID-1.1: Play App Signing

The app MUST use Play App Signing (Google-managed HSM) for production signing. Signing keys MUST NOT be stored on developer workstations or in source control.

**Testable:** Verify Play App Signing enrollment in Google Play Console. Verify no signing keys in repository.

### CODE-ANDROID-1.2: APK Signature Scheme v3+

The app MUST use APK Signature Scheme v3 or v4 for signing integrity. Legacy v1 (JAR signing) MUST NOT be the sole signing mechanism.

**Testable:** Run `apksigner verify --print-certs` on the APK. Verify v3 or v4 scheme is present.

### CODE-ANDROID-1.3: CI/CD Security Pipeline

The build pipeline MUST include:
- SAST (static analysis security testing) scans
- Dependency vulnerability scanning
- SBOM (Software Bill of Materials) generation
- Signing via HSM or secret vault (not local keystore files)

**Testable:** Review CI/CD configuration. Verify SAST, dependency scanning, and SBOM generation steps are present and passing.

### CODE-ANDROID-1.4: Reproducible Builds

The app SHOULD support reproducible builds where the same source code produces a bit-for-bit identical APK/AAB, enabling independent verification of build integrity.

**Testable:** Build from the same source twice. Compare output artifacts for identical hashes.

### CODE-ANDROID-1.5: Debug Build Separation

Release builds MUST have `android:debuggable="false"`. Debug-specific code, logging, and test endpoints MUST be removed or disabled in release builds via build variants or ProGuard/R8 rules.

**Testable:** Inspect release APK manifest for `debuggable` flag. Verify no debug endpoints are reachable.

## MASVS-CODE-2: Secure Third-Party Libraries

### CODE-ANDROID-2.1: SDK Supply Chain Vetting

All third-party SDKs MUST be vetted before integration:
- Review declared permissions and network behavior
- Monitor network traffic with mitmproxy or equivalent during testing
- Generate and review SBOM for transitive dependencies
- Document the decision to keep, replace, or remove each SDK

**Rationale:** Supply chain attacks through SDKs are a top mobile threat. SpinOK SDK affected 101 apps with 400M+ installs. Shai-Hulud npm worm compromised 500+ packages.

**Testable:** Verify SBOM exists and is current. Verify SDK vetting documentation exists. Verify no known vulnerable SDK versions via dependency scanning.

### CODE-ANDROID-2.2: SDK Permission Auditing

Third-party SDKs MUST NOT request permissions beyond what is necessary for their functionality. SDK permissions MUST be audited and documented.

**Testable:** Compare SDK-declared permissions against documented functionality. Verify no excessive permissions.

### CODE-ANDROID-2.3: SDK Consent Enforcement

Third-party SDKs MUST respect user consent signals. SDKs MUST NOT collect data before consent is confirmed. SDK data collection MUST be gated on the app's consent mechanism.

**Testable:** Monitor SDK network traffic before and after user consent. Verify no data transmission before consent.

### CODE-ANDROID-2.4: Dependency Vulnerability Monitoring

Dependencies MUST be continuously monitored for known CVEs. Vulnerable dependencies MUST be updated within a defined SLA (e.g., critical within 72 hours, high within 2 weeks).

**Testable:** Verify dependency scanning runs in CI/CD. Verify no dependencies with known critical/high CVEs beyond SLA.

## MASVS-CODE-3: Input Validation

### CODE-ANDROID-3.1: Untrusted Input Validation

All data from external sources (deep links, Intents, QR codes, NFC, clipboard, IPC) MUST be validated and sanitized before processing. The app MUST NOT trust any data received from other apps or external sources.

**Testable:** Send malformed/malicious data via each input vector. Verify the app handles it safely without crashes, injection, or unexpected behavior.

### CODE-ANDROID-3.2: WebView Input Sanitization

Data passed to WebViews (via `loadUrl`, `evaluateJavascript`, or JavaScript interfaces) MUST be sanitized to prevent XSS and injection attacks.

**Testable:** Inject JavaScript payloads via WebView input vectors. Verify no script execution occurs.

## MASVS-CODE-4: Security Update Readiness

### CODE-ANDROID-4.1: Minimum SDK Version

The app MUST set `minSdkVersion` to a supported Android version that receives security patches. Currently this means API 29 (Android 10) or higher.

**Testable:** Inspect `build.gradle` for `minSdkVersion`. Verify it meets the minimum requirement.

### CODE-ANDROID-4.2: Target SDK Currency

The app MUST target a recent SDK version as required by Google Play policy. The `targetSdkVersion` SHOULD be within one major version of the latest stable release.

**Testable:** Inspect `build.gradle` for `targetSdkVersion`. Verify it meets Google Play requirements.

### CODE-ANDROID-4.3: In-App Update Mechanism

The app SHOULD use the Play Core In-App Updates API to prompt users to update when security-critical updates are available, using the immediate update flow for critical fixes.

**Testable:** Verify In-App Updates API integration. Simulate available update. Verify prompt is displayed.
