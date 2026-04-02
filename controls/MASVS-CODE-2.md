# MASVS-CODE-2

## Control

The app has a mechanism for enforcing app updates.

## Description

When critical security vulnerabilities are discovered in a released Android app, users must be able to update promptly. Android provides the Google Play In-App Updates API for prompting users to update within the app. This control ensures the app can force or strongly encourage updates when security-critical issues are discovered.

## Android Sub-Requirements

### MASVS-CODE-2.1 — Implement In-App Update Checks

The app uses the Google Play In-App Updates API (`com.google.android.play:app-update`) to check for available updates and prompt the user. For critical security updates, the app uses the **immediate** update flow (full-screen, blocking) rather than the **flexible** (background download) flow.

**Android References:**
- `com.google.android.play:app-update` and `app-update-ktx` Gradle dependencies
- `AppUpdateManager.appUpdateInfo` — checks for available updates
- `AppUpdateType.IMMEDIATE` — full-screen blocking update
- `AppUpdateType.FLEXIBLE` — background download with user notification

### MASVS-CODE-2.2 — Implement Minimum Version Enforcement

The app checks a server-side minimum version endpoint on launch. If the running app version is below the minimum, the app blocks usage and directs the user to update. The server-side approach allows raising the minimum version in response to discovered vulnerabilities without requiring a client-side change.

**Rationale:** The In-App Updates API only works with Google Play. Server-side version enforcement works regardless of distribution channel and gives the security team the ability to force updates when a critical vulnerability is found.

### MASVS-CODE-2.3 — Handle Update Failures Gracefully

If the user declines or fails to update when a critical update is available, the app either:
- Restricts access to sensitive functionality while running the outdated version
- Re-prompts on next launch with increasing urgency
- Blocks usage entirely if the vulnerability is critical enough

The app does not crash, lose data, or degrade unexpectedly when handling update flows.

### MASVS-CODE-2.4 — Monitor Update Adoption

The app reports its version to the backend on each API call or session start, enabling the security team to monitor the distribution of app versions in the wild and track the adoption rate of critical security updates.

**Rationale:** Without version telemetry, the team has no visibility into how many users remain on vulnerable versions after a security update is released.
