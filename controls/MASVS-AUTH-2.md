# MASVS-AUTH-2

## Control

The app performs local authentication securely according to the platform best practices.

## Description

Many Android apps authenticate users locally via biometrics (fingerprint, face) or device credentials (PIN, pattern, password) using the BiometricPrompt API. Local authentication must be implemented correctly to prevent bypass — specifically, authentication results must be cryptographically bound to AndroidKeyStore operations rather than relying solely on callback success/failure booleans. This control ensures local authentication on Android is robust against instrumentation and tampering.

## Android Sub-Requirements

### MASVS-AUTH-2.1 — Use BiometricPrompt with CryptoObject Binding

Local biometric authentication uses `BiometricPrompt.authenticate(CryptoObject, ...)` (crypto-bound mode), not `BiometricPrompt.authenticate(...)` (result-only mode). The `CryptoObject` wraps a `Cipher`, `Signature`, or `Mac` initialized with a key from the AndroidKeyStore that requires user authentication.

**Rationale:** Result-only (convenience) authentication returns a success/failure boolean in the `AuthenticationCallback`. On a compromised device, an attacker can hook the callback and force a success result. Crypto-bound authentication ties biometric success to an actual cryptographic operation — the key is only usable after successful biometric verification by the TEE, which cannot be spoofed in software.

**Android References:**
- `BiometricPrompt.authenticate(BiometricPrompt.CryptoObject, BiometricPrompt.PromptInfo)`
- `BiometricPrompt.CryptoObject(cipher)` — wraps an initialized `Cipher`
- Key requirement: `KeyGenParameterSpec.Builder.setUserAuthenticationRequired(true)`

### MASVS-AUTH-2.2 — Require Strong Biometrics (Class 3)

The app requires `BiometricManager.Authenticators.BIOMETRIC_STRONG` (Class 3) for security-sensitive operations. Class 3 biometrics have hardware-backed security verification and meet the Android CDD's False Acceptance Rate requirements.

**Rationale:** `BIOMETRIC_WEAK` (Class 2) biometrics (e.g., some face unlock implementations) may not have hardware-backed matching and can be more easily spoofed. `BIOMETRIC_STRONG` ensures the biometric match is verified in the TEE/StrongBox.

**Android References:**
- `PromptInfo.Builder.setAllowedAuthenticators(BIOMETRIC_STRONG)`
- `PromptInfo.Builder.setAllowedAuthenticators(BIOMETRIC_STRONG | DEVICE_CREDENTIAL)` for fallback

### MASVS-AUTH-2.3 — Handle Biometric Enrollment Changes

Keys used for biometric authentication are created with `setInvalidatedByBiometricEnrollment(true)` (default). The app handles `KeyPermanentlyInvalidatedException` by re-enrolling the user (re-authenticating with primary credentials and generating a new key).

**Rationale:** When a new biometric is enrolled, an attacker who has device access could add their own fingerprint. Invalidating keys on enrollment change ensures the new biometric cannot access previously protected data without re-authentication.

### MASVS-AUTH-2.4 — Check Biometric Availability Before Prompting

The app checks `BiometricManager.canAuthenticate(authenticatorType)` before displaying the biometric prompt and handles all result codes:
- `BIOMETRIC_SUCCESS` — proceed with biometric auth
- `BIOMETRIC_ERROR_NONE_ENROLLED` — guide user to enroll biometrics
- `BIOMETRIC_ERROR_NO_HARDWARE` — fall back to device credential
- `BIOMETRIC_ERROR_HW_UNAVAILABLE` — retry later or fall back
- `BIOMETRIC_ERROR_SECURITY_UPDATE_REQUIRED` — prompt for security update

### MASVS-AUTH-2.5 — Implement Lockout on Failed Biometric Attempts

The app relies on the platform's biometric lockout (automatic after 5 failed attempts, with exponential backoff). The app does not implement custom retry logic that could bypass platform lockout.

**Rationale:** The Android platform enforces biometric lockout in the TEE/framework level. Custom retry logic that resets counters or allows unlimited attempts weakens this protection.

### MASVS-AUTH-2.6 — Use Protected Confirmation for High-Value Transactions

For the highest-value operations (financial transfers, legal agreements), the app uses `android.security.ConfirmationPrompt` (Protected Confirmation, API 28+) to obtain a TEE-signed confirmation that the user approved a specific action, even if the Android OS is fully compromised.

**Rationale:** Protected Confirmation renders the prompt in Trusted UI (TEE), signs the user's confirmation with a TEE-held key, and is verifiable server-side. This provides the strongest possible user confirmation on Android.

**Android References:**
- `ConfirmationPrompt.Builder(context).setPromptText(text).setExtraData(serverChallenge).build()`
- `ConfirmationCallback.onConfirmed(dataThatWasConfirmed)` — TEE-signed result
- Requires `FEATURE_TRUSTED_CONFIRMATION` system feature
