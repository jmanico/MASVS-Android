# MASVS-PRIVACY-ANDROID: Privacy Controls

Android-specific requirements for MASVS-PRIVACY-1 through MASVS-PRIVACY-4.

## MASVS-PRIVACY-1: Data Minimization

### PRIVACY-ANDROID-1.1: Scoped Storage Compliance

The app MUST use Scoped Storage (enforced since Android 10) and MUST NOT request `MANAGE_EXTERNAL_STORAGE` unless the app is a file manager or similar utility. Access to shared media MUST use the MediaStore API or Storage Access Framework.

**Testable:** Verify no `MANAGE_EXTERNAL_STORAGE` permission (unless justified). Verify MediaStore or SAF usage for shared media access.

### PRIVACY-ANDROID-1.2: Approximate Location

The app SHOULD use approximate location (`ACCESS_COARSE_LOCATION`) instead of precise location (`ACCESS_FINE_LOCATION`) when exact coordinates are not required for functionality. Available since Android 12.

**Testable:** Review location permission requests. Verify coarse location is used where precise is not necessary.

### PRIVACY-ANDROID-1.3: Selected Photos Access

On Android 14+, the app SHOULD use selected photos access to allow users to grant access to specific photos/videos rather than the entire media library.

**Testable:** Verify photo picker or selected access API usage. Verify the app functions with partial media access.

### PRIVACY-ANDROID-1.4: Minimal Permission Requests

The app MUST request only permissions that are necessary for its core functionality. Each permission MUST have a documented justification. Permissions MUST be requested at the point of use, not at app launch.

**Testable:** List all declared permissions. Verify each has a documented justification. Verify no permissions are requested at launch.

## MASVS-PRIVACY-2: Granular User Consent

### PRIVACY-ANDROID-2.1: Consent Before Collection

The app MUST obtain explicit user consent before collecting personal data. Consent MUST be granular (per data type/purpose), not bundled into a single accept-all prompt.

**Testable:** Install app fresh. Verify consent UI appears before any data collection. Verify granular consent options are available.

### PRIVACY-ANDROID-2.2: Play Data Safety Section

The app MUST maintain an accurate Google Play Data Safety section that reflects actual data collection, sharing, and security practices. Automated SDK analysis SHOULD be used to detect undeclared data collection by third-party SDKs.

**Testable:** Compare declared Data Safety practices with actual app behavior (network traffic analysis). Verify accuracy.

### PRIVACY-ANDROID-2.3: SDK Consent Gating

Third-party SDKs MUST NOT collect or transmit data before user consent is confirmed. The app MUST gate SDK initialization or data collection on the consent state.

**Testable:** Monitor network traffic before consent. Verify no SDK data transmissions before user grants consent.

## MASVS-PRIVACY-3: Anti-Tracking

### PRIVACY-ANDROID-3.1: Advertising ID Restrictions

The app MUST respect the user's choice to opt out of personalized advertising. If the user has reset or disabled their Advertising ID, the app MUST NOT use alternative identifiers for tracking.

**Testable:** Disable personalized ads in device settings. Verify app does not transmit advertising ID or alternative tracking identifiers.

### PRIVACY-ANDROID-3.2: No Fingerprinting

The app MUST NOT use device fingerprinting techniques (collecting hardware identifiers, installed app lists, sensor data, screen metrics) to create persistent user identifiers in violation of Google Play policy.

**Testable:** Monitor data collection. Verify no device fingerprinting data is assembled or transmitted.

### PRIVACY-ANDROID-3.3: Privacy Dashboard Compliance

On Android 12+, the app's permission usage MUST be consistent with its declared purposes. Users can review permission access history via the Privacy Dashboard.

**Testable:** Review Privacy Dashboard entries for the app. Verify permission usage is consistent with declared purposes and occurs only during expected user interactions.

## MASVS-PRIVACY-4: Data Deletion

### PRIVACY-ANDROID-4.1: Account and Data Deletion

The app MUST provide users with the ability to delete their account and associated personal data, as mandated by both Google Play and Apple App Store policies. The deletion MUST be:
- Accessible within the app (not just via email/support ticket)
- Complete (all server-side personal data removed or anonymized)
- Timely (processed within a documented timeframe)

**Testable:** Request account deletion via the app. Verify the option is accessible. Verify server-side data deletion within stated timeframe.

### PRIVACY-ANDROID-4.2: Local Data Cleanup

When a user deletes their account or logs out, the app MUST remove all locally stored personal data including:
- Keystore entries
- Encrypted storage files
- Database records
- Cached data and WebView storage
- Cookies

**Testable:** Delete account or logout. Inspect local storage directories. Verify no personal data remains.

### PRIVACY-ANDROID-4.3: Data Export

The app SHOULD provide users with the ability to export their personal data in a portable format, supporting data portability rights.

**Testable:** Request data export. Verify export is provided in a standard format (JSON, CSV, etc.) containing the user's personal data.
