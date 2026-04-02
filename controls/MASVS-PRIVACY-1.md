# MASVS-PRIVACY-1

## Control

The app minimizes access to sensitive data and resources.

## Description

Android apps should only request access to the data and system resources they absolutely need. Android's permission model has evolved significantly — from install-time grants to runtime permissions, one-time permissions, partial grants, and auto-revocation. This control ensures apps practice data minimization, request only necessary permissions, and enforce that third-party SDKs respect the same constraints.

## Android Sub-Requirements

### MASVS-PRIVACY-1.1 — Request Only Necessary Permissions

The app requests only the permissions that are strictly necessary for its core functionality. Every permission declared in `AndroidManifest.xml` has documented justification. Permissions that are only needed for optional features are requested when the feature is first used, not on app launch.

**Rationale:** Excessive permissions increase the attack surface and erode user trust. Google Play reviews apps for unnecessary permission requests.

### MASVS-PRIVACY-1.2 — Use the Most Restrictive Permission Available

When multiple permission levels can achieve the same goal, the app uses the least privileged option:
- `ACCESS_COARSE_LOCATION` instead of `ACCESS_FINE_LOCATION` when approximate location suffices
- `READ_MEDIA_IMAGES` (API 33+) instead of `READ_EXTERNAL_STORAGE` when only photos are needed
- `QUERY_ALL_PACKAGES` is avoided; use specific `<queries>` elements for package visibility

**Android References:**
- Android 12+: Users can grant approximate location separately from precise location
- Android 13+: Granular media permissions replace `READ_EXTERNAL_STORAGE`
- Android 14+: Partial photo/video access (user selects specific files)

### MASVS-PRIVACY-1.3 — Handle Permission Denials Gracefully

The app handles permission denials gracefully:
- Continues functioning with degraded features when non-essential permissions are denied
- Clearly explains why a permission is needed before requesting (via educational UI)
- Respects "Don't ask again" — directs the user to system settings if needed, without nagging
- Never blocks app launch solely due to a denied non-critical permission

### MASVS-PRIVACY-1.4 — Audit Third-Party SDK Data Collection

Every third-party SDK included in the app is audited for:
- What data it collects (analytics events, device identifiers, location, contacts)
- Whether it respects user consent signals (does it collect data before consent is granted?)
- Whether it transmits data to third-party servers
- Whether it can be configured to minimize or disable data collection

SDKs that collect data must be documented in the Google Play Data Safety Section.

**Rationale:** Third-party SDKs are the primary source of undisclosed data collection. The app developer is responsible for all data collection by included SDKs (OWASP Mobile Top 10 2024 M2).

### MASVS-PRIVACY-1.5 — Enforce SDK Consent Compliance

Third-party SDKs are initialized only after the user has granted consent. SDKs that ignore consent signals or begin data collection before consent is obtained are removed or replaced.

**Android References:**
- Initialize analytics/advertising SDKs only after consent flow completes
- Use SDK-specific consent APIs (e.g., Google Analytics consent mode, Firebase consent API)

### MASVS-PRIVACY-1.6 — Implement Scoped Storage Correctly

The app uses scoped storage (enforced since API 30):
- App-specific files in `getFilesDir()` / `getExternalFilesDir()` — no permission needed
- Shared media via `MediaStore` API with appropriate permissions
- User-selected files via Storage Access Framework (`ACTION_OPEN_DOCUMENT`)
- The app does not use `MANAGE_EXTERNAL_STORAGE` unless it is a file manager or similar utility, as this requires Google Play review

### MASVS-PRIVACY-1.7 — Minimize Background Data Access

The app does not access sensitive data (location, microphone, camera) in the background unless essential for core functionality (e.g., navigation, voice recording app). Background access permissions are requested separately with clear justification to the user.

**Android References:**
- `ACCESS_BACKGROUND_LOCATION` — separate permission since API 29
- Android 12+ privacy indicators show active camera/microphone access
- Background process restrictions (API 26+) limit background service execution
