# MASVS-NETWORK-2

## Control

The app performs identity pinning for all remote endpoints under the developer's control.

## Description

Certificate pinning restricts which certificates or public keys are trusted for specific domains, rather than trusting the entire system CA store. On Android, this is best implemented declaratively via the Network Security Configuration. Pinning protects against compromised CAs, rogue certificates, and sophisticated MITM attacks. This control ensures pinning is implemented correctly with proper backup pins and expiration handling.

## Android Sub-Requirements

### MASVS-NETWORK-2.1 — Implement Certificate Pinning via Network Security Config

The app configures certificate pinning in `network_security_config.xml` using `<pin-set>` for all remote endpoints under the developer's control. Pins are SPKI (SubjectPublicKeyInfo) SHA-256 hashes.

**Android References:**
```xml
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2027-01-01">
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

**Rationale:** Declarative pinning via Network Security Config is preferred over programmatic pinning (OkHttp `CertificatePinner`) because it is centralized, auditable, and managed independently of networking code.

### MASVS-NETWORK-2.2 — Include Backup Pins for Key Rotation

Every `<pin-set>` includes at least one backup pin that corresponds to a key not currently in production. This ensures the app can continue to function during emergency key rotation without requiring an app update.

**Rationale:** If the production key is compromised or its certificate expires and no backup pin is configured, the app becomes non-functional until users update to a version with new pins. Backup pins provide a recovery path.

### MASVS-NETWORK-2.3 — Set Pin Expiration Dates

Every `<pin-set>` includes an `expiration` attribute with a date that is far enough in the future to allow planned key rotation but not so far that stale pins persist indefinitely. After expiration, the platform falls back to standard certificate validation.

**Rationale:** Pinning without expiration creates a permanent bricking risk if all pinned keys are rotated and users have not updated the app. Expiration provides a safety valve.

### MASVS-NETWORK-2.4 — Pin Against SPKI Hashes, Not Certificate Hashes

Pins are computed against the SubjectPublicKeyInfo (SPKI) of the certificate, not the entire certificate. This allows certificate renewal without changing the pin, as long as the same public key is reused.

**Rationale:** Certificates are reissued regularly (e.g., Let's Encrypt every 90 days). Pinning the full certificate would break the app on every reissue. SPKI pinning survives certificate renewal if the key pair is preserved.

### MASVS-NETWORK-2.5 — Implement Pinning Failure Reporting

When a pin validation failure occurs, the app reports the failure to a server-side monitoring endpoint (with the domain, expected pins, and received certificate chain) for security incident detection. The app does not silently fall back to unpinned connections.

**Rationale:** Pin validation failures may indicate an active MITM attack. Reporting enables the security team to investigate and respond.

### MASVS-NETWORK-2.6 — Consider Certificate Transparency

For additional protection, the app verifies that server certificates include Signed Certificate Timestamps (SCTs) from Certificate Transparency logs. This provides an additional layer of assurance that certificates were publicly logged and not secretly issued.

**Rationale:** Certificate Transparency detects misissued or rogue certificates by requiring public logging. Combined with pinning, CT provides defense-in-depth against CA compromise.
