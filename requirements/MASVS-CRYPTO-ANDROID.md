# MASVS-CRYPTO-ANDROID: Cryptography

Android-specific requirements for MASVS-CRYPTO-1 and MASVS-CRYPTO-2.

## MASVS-CRYPTO-1: Strong Cryptography

### CRYPTO-ANDROID-1.1: Approved Algorithms Only

The app MUST use only current, strong cryptographic algorithms:
- **Symmetric encryption:** AES-256-GCM or ChaCha20-Poly1305
- **Hashing:** SHA-256, SHA-384, SHA-512, or SHA-3
- **Key derivation:** HKDF, PBKDF2 (with sufficient iterations), or Argon2
- **Asymmetric:** ECDSA (P-256 or P-384), RSA (2048+ bits with OAEP), Ed25519
- **HMAC:** HMAC-SHA256 or higher

The app MUST NOT use deprecated algorithms: DES, 3DES, RC4, MD5, SHA-1 (for security purposes), or Blowfish.

**Testable:** Static analysis identifies all cryptographic algorithm usage. No deprecated algorithms present.

### CRYPTO-ANDROID-1.2: Authenticated Encryption

All symmetric encryption MUST use authenticated encryption modes (GCM, CCM, or Poly1305). ECB mode MUST NOT be used. CBC mode without authentication MUST NOT be used for new implementations.

**Testable:** Static analysis verifies all `Cipher.getInstance()` calls use authenticated modes. No ECB or unauthenticated CBC usage.

### CRYPTO-ANDROID-1.3: Secure Random Number Generation

The app MUST use `java.security.SecureRandom` for all cryptographic random number generation. `java.util.Random`, `Math.random()`, or other non-cryptographic PRNGs MUST NOT be used for security-sensitive operations.

**Testable:** Static analysis confirms all security-related random generation uses `SecureRandom`.

### CRYPTO-ANDROID-1.4: Keystore-Supported Algorithms

When cryptographic operations involve Android Keystore, the app MUST use Keystore-supported algorithms: AES-GCM, HMAC-SHA256, ECDSA, or RSA. Key operations SHOULD remain within the Keystore (TEE/StrongBox) without exporting key material.

**Testable:** Verify key operations use `KeyStore.getInstance("AndroidKeyStore")` and key material is not extracted.

### CRYPTO-ANDROID-1.5: Post-Quantum Readiness

On Android 17+ (API 37), the app SHOULD evaluate use of HPKE hybrid cryptography (RFC 9180) for forward-looking quantum resistance where supported by the platform SPI.

**Testable:** Verify awareness and adoption plan for hybrid post-quantum key exchange on supported API levels.

## MASVS-CRYPTO-2: Key Management

### CRYPTO-ANDROID-2.1: No Hardcoded Keys

The app MUST NOT contain hardcoded encryption keys, API keys, or secrets in source code, resources, assets, or native libraries.

**Testable:** Static analysis (grep, semgrep, or equivalent) finds no hardcoded keys or secrets in APK contents.

### CRYPTO-ANDROID-2.2: Key Generation in Hardware

Cryptographic keys MUST be generated inside Android Keystore (TEE or StrongBox), not imported from software. This ensures key material never exists in app process memory.

**Testable:** Verify `KeyGenParameterSpec.Builder` is used with `setKeySize()` and key is generated via `KeyGenerator` or `KeyPairGenerator` with "AndroidKeyStore" provider.

### CRYPTO-ANDROID-2.3: Key Attestation

For high-assurance scenarios, the app SHOULD use Android Key Attestation to allow the server to verify that keys were generated in hardware and have specific security properties.

**Testable:** Verify attestation certificate chain is sent to server and validated against Google's root certificate.

### CRYPTO-ANDROID-2.4: Biometric-Gated Key Access

Keys protecting sensitive operations MUST be configured to require user authentication (biometric or device credential) before use, via `setUserAuthenticationRequired(true)` in `KeyGenParameterSpec`.

**Testable:** Attempt to use protected key without authentication. Verify `UserNotAuthenticatedException` is thrown.

### CRYPTO-ANDROID-2.5: Key Lifecycle Management

The app MUST implement proper key rotation procedures. Keys MUST have defined validity periods set via `setKeyValidityStart()` and `setKeyValidityEnd()` where appropriate. Expired keys MUST NOT be used for new encryption operations.

**Testable:** Verify key validity parameters are configured. Verify expired keys are rejected for new operations.

### CRYPTO-ANDROID-2.6: Symmetric Key Protection

If symmetric encryption keys must exist outside of Keystore (e.g., for database encryption), they MUST be wrapped (encrypted) by a Keystore-held key. The wrapping key MUST require user authentication.

**Testable:** Verify database encryption keys are wrapped by Keystore-held master keys. Verify wrapping key has authentication requirement.
