# MASVS-CRYPTO-2

## Control

The app performs key management according to industry best practices.

## Description

Even strong cryptographic algorithms are compromised by poor key management. On Android, the platform provides the AndroidKeyStore system, hardware-backed key storage (TEE and StrongBox), and key attestation for remote verification. This control ensures cryptographic keys are properly generated, stored, rotated, and destroyed throughout their lifecycle using Android-specific key management facilities.

## Android Sub-Requirements

### MASVS-CRYPTO-2.1 — Generate Keys Inside the AndroidKeyStore

Symmetric and asymmetric keys are generated directly inside the AndroidKeyStore using `KeyGenerator` or `KeyPairGenerator` with the `"AndroidKeyStore"` provider, not generated externally and then imported. This ensures the key material never exists in app process memory.

**Android References:**
- `KeyGenerator.getInstance("AES", "AndroidKeyStore")` with `KeyGenParameterSpec`
- `KeyPairGenerator.getInstance("EC", "AndroidKeyStore")` with `KeyGenParameterSpec`

### MASVS-CRYPTO-2.2 — Restrict Key Purpose

Each key is generated with the minimum necessary purposes via `KeyGenParameterSpec.Builder(alias, purpose)`. Encryption keys use `PURPOSE_ENCRYPT | PURPOSE_DECRYPT`. Signing keys use `PURPOSE_SIGN | PURPOSE_VERIFY`. Keys are never generated with `PURPOSE_ENCRYPT | PURPOSE_SIGN` combined.

**Rationale:** Multi-purpose keys increase the attack surface. A key used for both encryption and signing may be vulnerable to cross-protocol attacks.

### MASVS-CRYPTO-2.3 — Bind Keys to User Authentication

Keys protecting high-value data require user authentication before use. The app configures:
- `setUserAuthenticationRequired(true)`
- `setUserAuthenticationParameters(timeoutSeconds, AUTH_BIOMETRIC_STRONG)` or `AUTH_BIOMETRIC_STRONG | AUTH_DEVICE_CREDENTIAL`
- Appropriate timeout (0 for per-use authentication, or a short duration for time-bounded access)

**Rationale:** Without authentication binding, any code running as the app's UID can use the key. Authentication binding ensures the user is physically present.

### MASVS-CRYPTO-2.4 — Set Key Validity Periods

Keys have defined validity periods using `setKeyValidityStart()` and `setKeyValidityEnd()` or `setKeyValidityForOriginationEnd()` / `setKeyValidityForConsumptionEnd()` to enforce key rotation schedules.

**Rationale:** Unbounded key lifetimes increase the window for key compromise. Expired keys force the application to generate fresh key material.

### MASVS-CRYPTO-2.5 — Invalidate Keys on Biometric Enrollment Changes

Keys bound to biometric authentication are configured with `setInvalidatedByBiometricEnrollment(true)` (the default) so that adding new fingerprints or face enrollments invalidates existing keys.

**Rationale:** If a new biometric is enrolled (potentially by an attacker who has device access), previously created keys should not be usable with the new biometric.

### MASVS-CRYPTO-2.6 — Use Key Attestation for Remote Verification

For high-security use cases (e.g., payment, identity), the app generates a key pair with an attestation challenge from the server and sends the attestation certificate chain for server-side verification. The server:
1. Verifies the certificate chain signature
2. Confirms the root certificate matches the Google attestation root
3. Checks the `attestationSecurityLevel` is `TrustedEnvironment` or `StrongBox`
4. Validates the attestation extension for key properties
5. Checks certificate revocation status

**Android References:**
- `KeyGenParameterSpec.Builder.setAttestationChallenge(serverNonce)`
- Attestation certificate chain: `KeyStore.getCertificateChain(alias)`
- Google attestation root certificates (ECDSA P-384 root rotation begins February 2026)

### MASVS-CRYPTO-2.7 — Delete Keys When No Longer Needed

The app deletes cryptographic keys from the AndroidKeyStore when they are no longer needed (e.g., user logout, account deletion, key rotation) using `KeyStore.deleteEntry(alias)`.

**Rationale:** Residual keys in the KeyStore persist across app updates and reinstalls (if the app is not uninstalled). Orphaned keys are unnecessary attack surface.

### MASVS-CRYPTO-2.8 — Never Export Key Material from Hardware

The app does not attempt to export private or secret key material from hardware-backed storage. Keys generated in the AndroidKeyStore with `setIsStrongBoxBacked(true)` or hardware TEE backing are non-exportable by design.

**Rationale:** The entire security model of hardware-backed keys depends on the key material never leaving the secure element. Attempting to export (or wrapping with a software key) defeats this protection.
