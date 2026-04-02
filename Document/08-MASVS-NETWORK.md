# MASVS-NETWORK: Network Communication

## Overview

Android provides a robust, declarative system for network security through the Network Security Configuration (`network_security_config.xml`). This system controls cleartext traffic policy, certificate trust anchors, certificate pinning, and debug overrides — all without writing code. Combined with the platform's default TLS support (TLS 1.2 minimum since API 20, TLS 1.3 since API 29), Android apps have strong foundations for secure networking. This category ensures apps leverage these mechanisms correctly and do not undermine them with insecure custom implementations.

## Network Security Configuration Reference

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- Global: no cleartext, system CAs only -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>

    <!-- Per-domain: certificate pinning -->
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2027-01-01">
            <pin digest="SHA-256">primaryPinHash=</pin>
            <pin digest="SHA-256">backupPinHash=</pin>
        </pin-set>
    </domain-config>

    <!-- Debug-only: trust user CAs for proxy testing -->
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

## Android Version-Specific Behaviors

| API Level | Behavior |
|---|---|
| 24 (Android 7) | User-installed CAs no longer trusted by default |
| 28 (Android 9) | Cleartext traffic blocked by default |
| 29 (Android 10) | TLS 1.3 support enabled by default |
| 35 (Android 15) | TLS 1.0 and 1.1 disabled for targeting apps |

## Controls

| Control | Sub-Requirements | Focus |
|---|---|---|
| [MASVS-NETWORK-1](../controls/MASVS-NETWORK-1.md) | 8 sub-requirements | Secure network traffic configuration |
| [MASVS-NETWORK-2](../controls/MASVS-NETWORK-2.md) | 6 sub-requirements | Certificate/public key pinning |

## OWASP Mobile Top 10 2024 Mapping

- **M5 — Insecure Communication:** Directly addressed by both controls
- **M8 — Security Misconfiguration:** Addressed by MASVS-NETWORK-1.1, 1.3, 1.4
