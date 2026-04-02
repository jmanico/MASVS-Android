# MASVS-CRYPTO: Cryptography

## Overview

Cryptography is fundamental to protecting user data on Android, where physical device access is a realistic threat. Android provides the AndroidKeyStore for hardware-backed key storage, supports modern algorithms through `java.security` and `javax.crypto`, and offers Google Tink as a high-level cryptographic library. This category ensures the app uses strong cryptography with proper key management, leveraging Android's hardware security capabilities.

## Android Cryptographic Architecture

```
┌─────────────────────────────────────────┐
│            App Process                   │
│  ┌──────────┐  ┌───────────┐            │
│  │  Tink    │  │ java.     │            │
│  │  AEAD    │  │ security  │            │
│  └────┬─────┘  └─────┬─────┘            │
│       │               │                  │
│       └───────┬───────┘                  │
│               │                          │
│    ┌──────────▼──────────┐               │
│    │   AndroidKeyStore   │               │
│    │     Provider        │               │
│    └──────────┬──────────┘               │
└───────────────┼──────────────────────────┘
                │ Binder IPC
┌───────────────▼──────────────────────────┐
│         Keystore2 Daemon                 │
│    (keystore2, android.security)         │
└───────────────┬──────────────────────────┘
                │ HAL
┌───────────────▼──────────────────────────┐
│     Trusted Execution Environment        │
│  ┌────────────┐  ┌───────────────┐       │
│  │    TEE     │  │   StrongBox   │       │
│  │  (ARM      │  │  (Dedicated   │       │
│  │  TrustZone)│  │   Secure      │       │
│  │            │  │   Element)    │       │
│  └────────────┘  └───────────────┘       │
└──────────────────────────────────────────┘
```

## Approved Algorithm Reference

| Use Case | Algorithm | Android API |
|---|---|---|
| Symmetric encryption | AES-256-GCM | `KeyProperties.KEY_ALGORITHM_AES` + `BLOCK_MODE_GCM` |
| Asymmetric encryption | RSA-2048+ OAEP | `KEY_ALGORITHM_RSA` + `ENCRYPTION_PADDING_RSA_OAEP` |
| Signing | ECDSA P-256 | `KEY_ALGORITHM_EC` + `DIGEST_SHA256` |
| Signing | RSA-2048+ PSS | `KEY_ALGORITHM_RSA` + `SIGNATURE_PADDING_RSA_PSS` |
| HMAC | HMAC-SHA256 | `KEY_ALGORITHM_HMAC_SHA256` |
| Key agreement | ECDH P-256 | `KEY_ALGORITHM_EC` + `PURPOSE_AGREE_KEY` |
| PRNG | SecureRandom | `java.security.SecureRandom` (backed by `/dev/urandom`) |

## Controls

| Control | Sub-Requirements | Focus |
|---|---|---|
| [MASVS-CRYPTO-1](../controls/MASVS-CRYPTO-1.md) | 8 sub-requirements | Strong cryptographic algorithm usage |
| [MASVS-CRYPTO-2](../controls/MASVS-CRYPTO-2.md) | 8 sub-requirements | Key lifecycle management |

## OWASP Mobile Top 10 2024 Mapping

- **M10 — Insufficient Cryptography:** Directly addressed by both controls
- **M1 — Improper Credential Usage:** Addressed by MASVS-CRYPTO-1.4, MASVS-CRYPTO-2.1
