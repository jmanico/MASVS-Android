# MASVS-CRYPTO-1

## Control

The app employs current strong cryptography and uses it according to industry best practices.

## Description

Cryptography is critical for protecting user data on Android, where physical access to the device is a realistic threat model. Android provides platform cryptographic APIs (`java.security`, `javax.crypto`) and hardware-backed cryptographic operations via the AndroidKeyStore. This control ensures the app uses only strong, current algorithms with correct parameters and avoids the many pitfalls that lead to weak or broken cryptographic implementations on Android.

## Android Sub-Requirements

### MASVS-CRYPTO-1.1 — Use Approved Cryptographic Algorithms Only

The app uses only the following approved algorithms and modes:

| Use Case | Algorithm | Configuration |
|---|---|---|
| Symmetric encryption | AES-256 | GCM mode, no padding (`KeyProperties.BLOCK_MODE_GCM`, `ENCRYPTION_PADDING_NONE`) |
| Asymmetric encryption | RSA-2048+ | OAEP padding with SHA-256 (`ENCRYPTION_PADDING_RSA_OAEP`) |
| Digital signatures | ECDSA with P-256 | SHA-256 digest (`DIGEST_SHA256`) |
| Digital signatures | RSA-2048+ | PSS padding with SHA-256 (`SIGNATURE_PADDING_RSA_PSS`) |
| HMAC | HMAC-SHA256 or HMAC-SHA512 | `KEY_ALGORITHM_HMAC_SHA256` |
| Hashing (non-security) | SHA-256 or SHA-512 | `MessageDigest.getInstance("SHA-256")` |
| Key agreement | ECDH P-256 | `KEY_ALGORITHM_EC`, `PURPOSE_AGREE_KEY` |
| Password hashing | Argon2id or PBKDF2-HMAC-SHA256 | Minimum 600,000 iterations for PBKDF2 |

**Prohibited:** DES, 3DES, RC4, Blowfish, MD5, SHA-1 for signatures, ECB block mode, RSA with PKCS#1 v1.5 padding for encryption, any custom/proprietary cipher.

### MASVS-CRYPTO-1.2 — Never Use ECB Block Mode

The app never uses AES in ECB mode (`KeyProperties.BLOCK_MODE_ECB`). ECB encrypts identical plaintext blocks to identical ciphertext blocks, leaking patterns.

**Rationale:** ECB mode is the default in many `Cipher.getInstance()` calls when no mode is specified (provider-dependent). Always specify the full transformation string: `"AES/GCM/NoPadding"`.

### MASVS-CRYPTO-1.3 — Generate IVs and Nonces Securely

All initialization vectors (IVs) and nonces are generated using `SecureRandom` (which delegates to the kernel CSPRNG on Android). IVs are never hardcoded, reused across encryptions, or derived deterministically from predictable input.

**Rationale:** IV/nonce reuse in GCM mode is catastrophic — it enables key recovery. AES-GCM requires a unique 96-bit nonce for every encryption operation with the same key.

**Android References:**
- `SecureRandom` — backed by `/dev/urandom` on Android
- For AndroidKeyStore: the Keystore provider generates IVs internally; retrieve via `Cipher.getIV()` after `init()`

### MASVS-CRYPTO-1.4 — Do Not Hardcode Cryptographic Keys

The app does not contain hardcoded cryptographic keys, passwords, or secrets in source code, XML resources, `strings.xml`, `BuildConfig` fields, native libraries (`.so` files), or the `assets/` directory.

**Rationale:** Hardcoded keys are extractable from the APK via `apktool`, `jadx`, or `strings` on native libraries. Any secret embedded in the APK should be considered public.

### MASVS-CRYPTO-1.5 — Use the AndroidKeyStore Provider for Cryptographic Operations

All cryptographic operations on sensitive data use keys stored in the AndroidKeyStore (`KeyGenerator.getInstance(algorithm, "AndroidKeyStore")` or `KeyPairGenerator.getInstance(algorithm, "AndroidKeyStore")`).

**Rationale:** AndroidKeyStore keys are bound to the device's TEE or StrongBox secure element. The key material never enters the app's process memory and cannot be extracted, even on rooted devices (for hardware-backed keys).

**Android References:**
- `KeyGenerator.getInstance("AES", "AndroidKeyStore")`
- `KeyPairGenerator.getInstance("EC", "AndroidKeyStore")`
- `KeyStore.getInstance("AndroidKeyStore")`

### MASVS-CRYPTO-1.6 — Verify Hardware-Backed Key Storage

The app verifies that cryptographic keys are hardware-backed by checking `KeyInfo.getSecurityLevel()` returns `SECURITY_LEVEL_TRUSTED_ENVIRONMENT` or `SECURITY_LEVEL_STRONGBOX`. If hardware backing is unavailable, the app degrades gracefully with appropriate risk acceptance.

**Android References:**
- `KeyFactory.getInstance(key.getAlgorithm(), "AndroidKeyStore").getKeySpec(key, KeyInfo.class)`
- `KeyInfo.getSecurityLevel()` (API 31+)
- `KeyInfo.isInsideSecureHardware()` (deprecated but available on older APIs)

### MASVS-CRYPTO-1.7 — Use SecureRandom for All Random Number Generation

All security-sensitive random values (session tokens, nonces, salts, key material) are generated using `java.security.SecureRandom`. The app does not use `java.util.Random`, `Math.random()`, or `System.currentTimeMillis()` for security-sensitive purposes.

**Rationale:** `java.util.Random` uses a predictable linear congruential generator. `SecureRandom` on Android delegates to the kernel CSPRNG (`/dev/urandom`).

### MASVS-CRYPTO-1.8 — Avoid Insecure Cipher.getInstance() Calls

Every `Cipher.getInstance()` call specifies the full transformation string including algorithm, mode, and padding (e.g., `"AES/GCM/NoPadding"`). The app never calls `Cipher.getInstance("AES")` without specifying mode and padding, as the default varies by provider and may select ECB.

**Rationale:** The default transformation for `Cipher.getInstance("AES")` is provider-dependent. On some Android versions and providers, it defaults to `AES/ECB/PKCS5Padding`, which is insecure.
