# MASVS-AUTH-3

## Control

The app secures sensitive operations with additional authentication.

## Description

Certain operations within an app require additional authentication beyond the initial login — changing account settings, authorizing payments, accessing highly sensitive data, or performing irreversible actions. On Android, this step-up authentication can leverage BiometricPrompt, the Credential Manager, server-side re-authentication, or Protected Confirmation. This control ensures that sensitive operations are gated by appropriate additional authentication.

## Android Sub-Requirements

### MASVS-AUTH-3.1 — Gate Sensitive Operations with Step-Up Authentication

The app requires re-authentication (biometric, device credential, or server-side) before performing sensitive operations including:
- Changing passwords or security settings
- Modifying payment methods or making financial transactions
- Exporting or deleting account data
- Granting new device or session access
- Viewing highly sensitive PII (SSN, full account numbers)

**Rationale:** The initial authentication may have been performed minutes or hours ago. Step-up authentication ensures the current user is authorized for the specific sensitive operation.

### MASVS-AUTH-3.2 — Bind Step-Up Authentication to Cryptographic Operations

Step-up re-authentication uses crypto-bound BiometricPrompt (see MASVS-AUTH-2.1) where the cryptographic operation (e.g., signing a transaction request) can only complete after successful biometric verification. The server verifies the signed request.

**Rationale:** Without crypto-binding, a compromised app can bypass the step-up prompt and directly call the sensitive API. Crypto-binding ensures the server receives proof that biometric authentication occurred.

### MASVS-AUTH-3.3 — Use Per-Use Key Authentication for Critical Operations

For the most critical operations, the AndroidKeyStore key requires per-use authentication (`setUserAuthenticationParameters(0, AUTH_BIOMETRIC_STRONG)` — zero timeout means every use requires authentication). The key cannot be used in a time-bounded window after a single authentication.

**Rationale:** Time-bounded authentication (e.g., 30-second window) allows multiple operations after a single biometric prompt. Per-use authentication ensures each critical operation individually requires biometric verification.

### MASVS-AUTH-3.4 — Implement Server-Side Step-Up Verification

The server independently verifies that step-up authentication was performed before executing the sensitive operation. Client-side authentication gates alone are insufficient — the server must verify a cryptographic proof (signed challenge, attestation) or re-authenticate via a server-side mechanism (OTP, re-entered password).

**Rationale:** Client-side authentication checks can be bypassed by modifying the app, hooking callbacks, or directly calling API endpoints. Server-side verification is the authoritative enforcement point.

### MASVS-AUTH-3.5 — Support Multi-Factor Authentication

For high-risk applications (financial, healthcare, enterprise), the app supports multi-factor authentication combining:
- Something the user knows (password/PIN)
- Something the user has (device with passkey, TOTP authenticator)
- Something the user is (biometric via BiometricPrompt)

**Rationale:** Single-factor authentication can be compromised through individual vector (stolen password, spoofed biometric). MFA requires compromise of multiple independent factors.

### MASVS-AUTH-3.6 — Rate-Limit Step-Up Authentication Attempts

The app and server implement rate limiting and lockout for step-up authentication failures. After a configured number of failed step-up attempts, the session is terminated and full re-authentication is required.

**Rationale:** Without rate limiting, an attacker who has session access can brute-force the step-up authentication. Session termination after repeated failures limits this attack.
