# MASVS-NETWORK-ANDROID: Network Communication

Android-specific requirements for MASVS-NETWORK-1 and MASVS-NETWORK-2.

## MASVS-NETWORK-1: Secure Network Traffic

### NETWORK-ANDROID-1.1: Network Security Configuration

The app MUST include a Network Security Configuration (NSC) file that:
- Sets `cleartextTrafficPermitted="false"` as the base configuration
- Does NOT include `<debug-overrides>` with user-added CAs in release builds
- Does NOT include per-domain cleartext exceptions unless absolutely necessary and documented

**Testable:** Inspect `res/xml/network_security_config.xml`. Verify `cleartextTrafficPermitted="false"` in base-config. Verify no debug-overrides in release APK.

### NETWORK-ANDROID-1.2: TLS 1.3 Preferred

The app MUST support TLS 1.2 as minimum and SHOULD prefer TLS 1.3 (available since Android 10/API 29). TLS 1.0 and 1.1 MUST NOT be used.

**Testable:** Capture TLS handshake with proxy. Verify negotiated TLS version is 1.2 or 1.3. No successful connections on TLS 1.0/1.1.

### NETWORK-ANDROID-1.3: Fail-Closed on TLS Failure

If a TLS connection fails, the app MUST NOT fall back to unencrypted HTTP. All network communication MUST fail closed — no plaintext fallback under any circumstance.

**Testable:** Simulate TLS failure (invalid cert, handshake timeout). Verify app does not retry over HTTP. Verify appropriate error is displayed.

### NETWORK-ANDROID-1.4: No Trust Manager Override

The app MUST NOT implement custom `TrustManager` or `HostnameVerifier` that bypasses certificate validation. `X509TrustManager.checkServerTrusted()` MUST NOT be implemented as a no-op.

**Testable:** Static analysis for custom `TrustManager`, `HostnameVerifier`, or `SSLSocketFactory` implementations. Verify none accept all certificates.

### NETWORK-ANDROID-1.5: Secure API Communication

- All API requests MUST use HTTPS
- Authentication tokens MUST be sent in the `Authorization` header, never in URL query parameters
- Sensitive API responses MUST include `Cache-Control: no-store` headers
- Request signing (HMAC or DPoP) SHOULD be used for sensitive operations

**Testable:** Inspect all API traffic. Verify HTTPS-only, header-based auth, no token leakage in URLs, appropriate cache headers.

### NETWORK-ANDROID-1.6: Android 17 Cleartext Deprecation

For apps targeting Android 17 (API 37+), the `usesCleartextTraffic` manifest attribute is deprecated. The app MUST use Network Security Configuration exclusively for traffic policy.

**Testable:** Verify apps targeting SDK 37+ use NSC instead of manifest attribute for cleartext policy.

## MASVS-NETWORK-2: Certificate Transparency and Pinning

### NETWORK-ANDROID-2.1: Certificate Transparency

On Android 17+, Certificate Transparency (CT) is enabled by default. For earlier Android versions, the app SHOULD verify CT compliance of server certificates where feasible.

**Rationale:** The industry has moved from certificate pinning to Certificate Transparency. Android 14 removed `pin-set` support from NSC. CT provides better security with lower operational risk than pinning (no risk of bricking the app on certificate rotation).

**Testable:** On Android 17+, verify CT enforcement is active (not disabled). On earlier versions, verify CT compliance checking if implemented.

### NETWORK-ANDROID-2.2: Certificate Pinning (Legacy/High-Security Only)

Certificate pinning SHOULD NOT be used for most applications due to operational risks (certificate rotation failures, CA root expiry incidents like Let's Encrypt 2021). Pinning is acceptable ONLY for very high-security apps that have mature certificate rotation procedures and incident response plans.

If pinning is used, it MUST:
- Pin to the public key (SPKI), not the full certificate
- Include at least one backup pin
- Implement a reporting mechanism for pin failures
- Have a documented rotation procedure

**Testable:** If pinning is present, verify SPKI pinning, backup pin, and failure reporting. If pinning is absent, verify CT compliance per NETWORK-ANDROID-2.1.

### NETWORK-ANDROID-2.3: Full Certificate Chain Validation

The app MUST validate the full certificate chain to a trusted root CA. Self-signed certificates MUST NOT be accepted in production builds.

**Testable:** Attempt connection with self-signed certificate. Verify connection is rejected. Verify full chain validation via proxy inspection.
