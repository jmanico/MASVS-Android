# MASVS-AUTH: Authentication and Authorization

## Overview

Android apps authenticate users through remote protocols (OAuth 2.0, OpenID Connect, passkeys) and local mechanisms (BiometricPrompt, device credentials). Both paths have Android-specific implementation requirements. This category ensures authentication is implemented securely on the Android client — using the Credential Manager API, crypto-bound biometrics, and proper token management — while recognizing that server-side enforcement is the authoritative control.

## Android Authentication Architecture

```
┌────────────────────────────────────────────┐
│                  User                       │
│         ┌──────────┐                        │
│         │ Biometric│ ← Fingerprint/Face     │
│         └────┬─────┘                        │
└──────────────┼──────────────────────────────┘
               │
┌──────────────▼──────────────────────────────┐
│          BiometricPrompt                     │
│   (CryptoObject binding to KeyStore key)     │
│                                              │
│   ┌────────────────────────────────────┐     │
│   │      Credential Manager API        │     │
│   │  ┌──────┐ ┌────────┐ ┌─────────┐  │     │
│   │  │Passkey│ │Password│ │Federated│  │     │
│   │  │(FIDO2)│ │ (auto) │ │(Google) │  │     │
│   │  └──┬───┘ └───┬────┘ └────┬────┘  │     │
│   └─────┼─────────┼───────────┼────────┘     │
└─────────┼─────────┼───────────┼──────────────┘
          │         │           │
          ▼         ▼           ▼
    AndroidKeyStore  Server    Identity
    (TEE/StrongBox)            Provider
```

## Key Android APIs

- **Credential Manager** (`androidx.credentials`) — Unified API for passkeys, passwords, federated sign-in
- **BiometricPrompt** (`androidx.biometric`) — Biometric authentication with Class 3 (strong) support
- **Protected Confirmation** (`android.security.ConfirmationPrompt`) — TEE-signed user confirmation
- **AndroidKeyStore** — Hardware-backed keys bound to user authentication

## Controls

| Control | Sub-Requirements | Focus |
|---|---|---|
| [MASVS-AUTH-1](../controls/MASVS-AUTH-1.md) | 6 sub-requirements | Secure authentication protocols and token management |
| [MASVS-AUTH-2](../controls/MASVS-AUTH-2.md) | 6 sub-requirements | Secure local (biometric/credential) authentication |
| [MASVS-AUTH-3](../controls/MASVS-AUTH-3.md) | 6 sub-requirements | Step-up authentication for sensitive operations |

## OWASP Mobile Top 10 2024 Mapping

- **M3 — Insecure Authentication/Authorization:** Directly addressed by all three controls
- **M1 — Improper Credential Usage:** Addressed by MASVS-AUTH-1.3, 1.4, 1.5
