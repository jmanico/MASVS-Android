# MASVS-PLATFORM: Platform Interaction

## Overview

Android's component model — Activities, Services, Broadcast Receivers, and Content Providers — combined with the Intent system, WebViews, and deep links, creates a rich but complex attack surface. Every exported component is an entry point accessible to any app on the device. WebViews bridge web content to native code. Deep links expose the app to the internet. This category ensures all platform interactions are secured against the Android-specific attack patterns that exploit these mechanisms.

## Android IPC Attack Surface

| Component | Attack Vector | Key Risk |
|---|---|---|
| Exported Activity | Intent injection, task hijacking (StrandHogg) | Unauthorized access, credential phishing |
| Content Provider | SQL injection, path traversal, data leakage | Data theft, arbitrary file read |
| Broadcast Receiver | Broadcast theft, spoofed broadcasts | Data interception, unauthorized actions |
| Service / Bound Service | Unauthorized service invocation | Privilege escalation, data access |
| PendingIntent | Mutable intent hijacking | Redirect operations to malicious targets |
| Deep Link | Parameter injection, unvalidated navigation | Arbitrary activity launch, XSS |
| WebView | JavaScript bridge abuse, file access, XSS | Native code execution, data theft |
| App Links | Link hijacking (if not verified) | Credential interception |

## Android Version-Specific Protections

| API Level | Protection |
|---|---|
| 17 | `MODE_WORLD_READABLE`/`WRITABLE` deprecated |
| 24 | User CAs not trusted; `FileProvider` recommended |
| 28 | Cleartext blocked; Protected Confirmation |
| 30 | Package visibility restrictions |
| 31 | Explicit `exported` required; `FLAG_IMMUTABLE` for PendingIntent |
| 33 | `RECEIVER_NOT_EXPORTED` flag |
| 35 | Safer intent matching; screen recording detection |
| 36 | Safer Intents v2; intent redirection hardening; local network permission |

## Controls

| Control | Sub-Requirements | Focus |
|---|---|---|
| [MASVS-PLATFORM-1](../controls/MASVS-PLATFORM-1.md) | 10 sub-requirements | IPC mechanism security |
| [MASVS-PLATFORM-2](../controls/MASVS-PLATFORM-2.md) | 7 sub-requirements | WebView security |
| [MASVS-PLATFORM-3](../controls/MASVS-PLATFORM-3.md) | 6 sub-requirements | UI security (tapjacking, screenshots, task hijacking) |

## OWASP Mobile Top 10 2024 Mapping

- **M4 — Insufficient Input/Output Validation:** Addressed by MASVS-PLATFORM-1.4, 1.5, 1.7, 2.2, 2.4
- **M8 — Security Misconfiguration:** Addressed by MASVS-PLATFORM-1.1, 1.2, 2.1, 2.3, 2.5
- **M3 — Insecure Authentication/Authorization:** Addressed by MASVS-PLATFORM-3.1, 3.5
