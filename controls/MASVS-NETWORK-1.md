# MASVS-NETWORK-1

## Control

The app secures all network traffic according to the current best practices.

## Description

Ensuring data confidentiality and integrity for all network communication is critical for any Android app. Android provides the Network Security Configuration system for declarative TLS policy, enforces cleartext traffic restrictions by default since API 28, and offers TLS 1.3 support. This control ensures the app correctly configures and maintains secure network connections for all traffic, using Android-specific mechanisms.

## Android Sub-Requirements

### MASVS-NETWORK-1.1 — Disable Cleartext Traffic

The app blocks all cleartext (HTTP) traffic by configuring `cleartextTrafficPermitted="false"` in `network_security_config.xml` or verifying the default (false since `targetSdkVersion >= 28`). Per-domain exceptions are only used for documented, justified cases (e.g., localhost development).

**Android References:**
```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```
- `android:networkSecurityConfig="@xml/network_security_config"` in `<application>`
- `StrictMode.VmPolicy.Builder.detectCleartextNetwork()` for development-time detection

### MASVS-NETWORK-1.2 — Enforce TLS 1.2 as Minimum, Prefer TLS 1.3

The app uses TLS 1.2 as the minimum protocol version. TLS 1.0 and 1.1 are not permitted (Android 15+ disables them by default for apps targeting API 35+). TLS 1.3 is preferred where supported.

**Rationale:** TLS 1.0 and 1.1 have known vulnerabilities (BEAST, POODLE, Lucky13). TLS 1.3 provides improved performance (1-RTT handshake) and security (no vulnerable cipher suites).

**Android References:**
- `SSLContext.getInstance("TLSv1.3")` for explicit protocol selection
- `OkHttpClient.Builder().connectionSpecs(listOf(ConnectionSpec.MODERN_TLS))`

### MASVS-NETWORK-1.3 — Do Not Override TrustManager or HostnameVerifier

The app does not implement custom `X509TrustManager` that accepts all certificates, custom `HostnameVerifier` that accepts all hostnames, or custom `SSLSocketFactory` that disables validation. These patterns are never present in production code, even behind feature flags or debug conditions.

**Rationale:** Trust-all and hostname-skip implementations completely negate TLS security, enabling any network attacker to intercept all traffic. These are the most common Android network security vulnerabilities.

**Prohibited patterns:**
```java
// NEVER DO THIS
TrustManager[] trustAll = new TrustManager[]{ new X509TrustManager() {
    public void checkClientTrusted(X509Certificate[] chain, String type) {}
    public void checkServerTrusted(X509Certificate[] chain, String type) {}
    public X509Certificate[] getAcceptedIssuers() { return new X509Certificate[0]; }
}};

// NEVER DO THIS
HostnameVerifier acceptAll = (hostname, session) -> true;
```

### MASVS-NETWORK-1.4 — Do Not Trust User-Installed CA Certificates in Production

The `network_security_config.xml` does not include `<certificates src="user" />` in the `<base-config>` trust anchors for release builds. Only system CAs are trusted.

**Rationale:** User-installed CA certificates enable MITM proxying of all app traffic by any party that can install a certificate on the device (MDM, employer, attacker with device access). Since API 24, user CAs are not trusted by default.

**Android References:**
- Default behavior since API 24 (`targetSdkVersion >= 24`): only system CAs trusted
- Debug-only exception:
```xml
<debug-overrides>
    <trust-anchors>
        <certificates src="user" />
    </trust-anchors>
</debug-overrides>
```

### MASVS-NETWORK-1.5 — Validate Server Certificates Properly

The app relies on the platform's default certificate validation chain (system trust store, Network Security Config). If the app performs any custom certificate handling, it uses `CertPathValidator` with the platform's default `TrustAnchor` set and does not bypass validation for any certificate error.

### MASVS-NETWORK-1.6 — Use Network Security Config for All TLS Policy

All TLS configuration (trust anchors, cleartext policy, certificate pinning, debug overrides) is managed through `res/xml/network_security_config.xml`, not through programmatic SSL/TLS configuration in code. This provides a declarative, auditable, centralized security policy.

**Rationale:** Declarative configuration in Network Security Config is easier to audit, less error-prone than programmatic configuration, and survives code changes that might accidentally weaken TLS settings.

### MASVS-NETWORK-1.7 — Secure WebSocket and Non-HTTP Connections

If the app uses WebSocket, gRPC, MQTT, or other non-HTTP protocols, these connections also use TLS (WSS, gRPC with TLS, MQTTS). The same TLS requirements (minimum version, certificate validation, no cleartext) apply to all network protocols, not just HTTP.

**Rationale:** Network security requirements apply to all transport protocols. WebSocket over `ws://` or MQTT over plaintext are equally vulnerable to interception as HTTP.

### MASVS-NETWORK-1.8 — Consider DNS Security

The app is aware that standard DNS queries are unencrypted and can be intercepted or spoofed. For high-security applications, the app uses DNS-over-HTTPS (DoH) via `OkHttp` or `android.net.DnsResolver`, or relies on the user's Private DNS configuration (DoT, available since Android 9).

**Rationale:** DNS queries reveal which servers the app communicates with and can be spoofed to redirect traffic to attacker-controlled servers, even when TLS is used (if certificate validation is then bypassed).
