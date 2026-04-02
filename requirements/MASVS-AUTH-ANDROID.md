# MASVS-AUTH-ANDROID: Authentication and Authorization

Android-specific requirements for MASVS-AUTH-1, MASVS-AUTH-2, and MASVS-AUTH-3.

## MASVS-AUTH-1: Secure Authentication Protocols

### AUTH-ANDROID-1.1: System Browser for OAuth

OAuth and OpenID Connect flows MUST use the system browser via Chrome Custom Tabs (`CustomTabsIntent`) or equivalent. Embedded `WebView` MUST NOT be used for OAuth flows per RFC 8252.

**Testable:** Verify OAuth login launches a Custom Tab (system browser), not a WebView. Inspect for `CustomTabsIntent.Builder` usage.

### AUTH-ANDROID-1.2: PKCE Requirement

All OAuth 2.0 authorization code flows MUST use Proof Key for Code Exchange (PKCE) with S256 challenge method.

**Testable:** Capture OAuth authorization request. Verify `code_challenge` and `code_challenge_method=S256` parameters are present.

### AUTH-ANDROID-1.3: Token Storage Security

- Access tokens: stored in memory or encrypted storage, 5-15 minute lifetime
- Refresh tokens: stored in Android Keystore, rotated on every use
- Tokens MUST NOT be stored in `SharedPreferences`, query parameters, or logged

**Testable:** Inspect storage locations for tokens. Verify refresh token rotation (new token issued on each refresh). Verify access token TTL.

### AUTH-ANDROID-1.4: DPoP Token Binding

For high-security apps, access tokens SHOULD be sender-constrained using DPoP (RFC 9449). DPoP proof keys SHOULD be hardware-bound using Android Keystore.

**Testable:** Verify DPoP proof header in API requests. Verify proof key originates from Keystore.

### AUTH-ANDROID-1.5: Server-Side Token Revocation on Logout

On logout, the app MUST call the server-side token revocation endpoint, clear all tokens from Keystore and encrypted storage, and clear WebView cookies and cache.

**Testable:** Capture logout network traffic for revocation call. Verify local token storage is empty post-logout. Verify WebView data cleared.

### AUTH-ANDROID-1.6: Credential Manager for Passkeys

The app SHOULD use the Android Credential Manager API (available since Android 9 via Play Services) for passkey (FIDO2/WebAuthn) authentication. Passkey private keys MUST be stored in hardware (Keystore).

**Testable:** Verify `CredentialManager.create()` / `CredentialManager.get()` API usage. Verify WebAuthn registration/authentication flows complete successfully.

### AUTH-ANDROID-1.7: Session Concurrency Controls

The app SHOULD implement concurrent session detection, limiting active sessions (e.g., max 3 devices) and notifying users via push notification when new sessions are created.

**Testable:** Authenticate on multiple devices beyond limit. Verify oldest session is invalidated or user is notified.

## MASVS-AUTH-2: Secure Local Authentication

### AUTH-ANDROID-2.1: BiometricPrompt with Class 3 Biometrics

Local biometric authentication MUST use `BiometricPrompt` API with `BIOMETRIC_STRONG` (Class 3) authenticators. Deprecated APIs (`FingerprintManager`) MUST NOT be used.

**Testable:** Verify `BiometricPrompt` usage with `setAllowedAuthenticators(BIOMETRIC_STRONG)`. No `FingerprintManager` imports.

### AUTH-ANDROID-2.2: CryptoObject Binding

Biometric authentication MUST bind to a cryptographic operation via `BiometricPrompt.CryptoObject`. The biometric MUST gate a Keystore key operation (sign, encrypt, decrypt), not merely a UI callback.

**Rationale:** Without CryptoObject binding, the biometric callback can be hooked and bypassed via Frida or similar tools. The biometric must gate a cryptographic operation, not just a UI flag.

**Testable:** Verify `BiometricPrompt.authenticate(CryptoObject, ...)` is called (not the overload without CryptoObject). Verify CryptoObject wraps a Keystore-backed Cipher, Signature, or Mac.

### AUTH-ANDROID-2.3: Biometric Invalidation on Enrollment Change

Keys gated by biometric authentication MUST be invalidated when biometric enrollment changes (new fingerprint added). Use `setInvalidatedByBiometricEnrollment(true)` in `KeyGenParameterSpec`.

**Testable:** Add a new fingerprint. Attempt to use the biometric-gated key. Verify the key is invalidated and re-authentication is required.

## MASVS-AUTH-3: Additional Authentication for Sensitive Operations

### AUTH-ANDROID-3.1: Step-Up Authentication

Sensitive operations (e.g., financial transactions, profile changes, password resets) MUST require step-up authentication via biometric confirmation, PIN re-entry, or MFA code even within an active session.

**Testable:** Attempt sensitive operation within active session. Verify additional authentication prompt is displayed before operation proceeds.

### AUTH-ANDROID-3.2: Background Session Handling

When the app moves to background, it MUST clear sensitive data from memory and invalidate the session UI. The refresh token MAY be retained for session resumption, but the user MUST re-authenticate for sensitive operations after returning from background.

**Testable:** Background the app, return after configurable timeout. Verify sensitive screens require re-authentication.
