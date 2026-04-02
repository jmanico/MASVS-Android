# MASVS-AUTH-1

## Control

The app uses secure authentication and authorization protocols and follows the relevant best practices.

## Description

Most Android apps connect to remote endpoints that require user authentication and enforce authorization. While server-side enforcement is critical, the Android client must also implement authentication flows securely — protecting tokens in transit and at rest, using platform-provided credential management, and avoiding implementation patterns that expose credentials to interception or replay. This control covers secure use of authentication protocols on the Android client side.

## Android Sub-Requirements

### MASVS-AUTH-1.1 — Use the Credential Manager API for Authentication

The app uses `androidx.credentials.CredentialManager` as the unified authentication surface for passkeys (FIDO2 WebAuthn), passwords, and federated sign-in. The app does not use the deprecated FIDO2 library (`com.google.android.gms:play-services-fido`) or custom authentication UI for standard credential types.

**Rationale:** Credential Manager provides a single, consistent, and secure authentication flow that integrates with the platform's autofill, biometric prompt, and password manager infrastructure. It reduces implementation surface area and supports passkeys natively.

**Android References:**
- `androidx.credentials.CredentialManager`
- `CreatePublicKeyCredentialRequest` — passkey registration
- `GetPublicKeyCredentialOption` — passkey authentication
- `GetPasswordOption` — password retrieval

### MASVS-AUTH-1.2 — Prefer Passkeys Over Passwords

The app supports passkeys (FIDO2 discoverable credentials) as the primary authentication mechanism. Passkeys are synced via Google Password Manager and provide phishing-resistant, password-free authentication.

**Rationale:** Passkeys eliminate password reuse, phishing, and credential stuffing attacks. On Android 9+, passkeys are hardware-backed (private key in AndroidKeyStore/TEE) and synced across devices.

### MASVS-AUTH-1.3 — Store Authentication Tokens Securely

OAuth tokens, session tokens, JWTs, and API keys are stored in the AndroidKeyStore (encrypted with a hardware-backed key) or in encrypted DataStore/Tink storage. Tokens are never stored in plaintext SharedPreferences, databases without encryption, or external storage.

**Rationale:** Stolen tokens allow session hijacking and unauthorized API access. Hardware-backed encrypted storage ensures tokens survive only device compromise with root + TEE bypass.

### MASVS-AUTH-1.4 — Implement Token Refresh and Expiration

The app implements short-lived access tokens with refresh token rotation. Expired tokens are not reused. The refresh token is stored with stronger protection (biometric-bound key) than the access token.

**Rationale:** Short-lived tokens limit the window of compromise. Refresh token rotation detects token theft (when both parties attempt to use the same refresh token).

### MASVS-AUTH-1.5 — Clear Authentication State on Logout

On user logout, the app:
1. Revokes the refresh token server-side
2. Deletes all local tokens from encrypted storage
3. Clears any cached credentials from `CredentialManager`
4. Clears WebView cookies and session data if web-based auth was used
5. Navigates to the login screen and clears the back stack

**Rationale:** Incomplete logout leaves residual authentication state that can be reused by subsequent users of the device or by malware.

### MASVS-AUTH-1.6 — Do Not Implement Custom TLS or Certificate Validation in Auth Flows

Authentication flows use the platform's default `HttpsURLConnection` or `OkHttp` TLS implementation. The app does not implement custom `X509TrustManager`, `HostnameVerifier`, or `SSLSocketFactory` that weakens validation during authentication.

**Rationale:** Custom TLS implementations in authentication flows frequently introduce trust-all or hostname-skip vulnerabilities that enable credential interception via MITM.
