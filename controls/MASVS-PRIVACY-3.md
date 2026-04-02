# MASVS-PRIVACY-3

## Control

The app is transparent about data collection and usage.

## Description

Users have the right to understand how their data is collected, stored, processed, and shared. On Android, Google Play requires a Data Safety Section that declares all data practices. This control ensures the app provides clear, accurate, and complete transparency about its data practices — both to the platform (Data Safety Section) and to the user (in-app disclosures).

## Android Sub-Requirements

### MASVS-PRIVACY-3.1 — Complete and Accurate Google Play Data Safety Section

The app's Google Play Data Safety Section accurately declares:
- All data types collected (location, contacts, financial, health, messages, photos, identifiers, etc.)
- Whether each data type is collected, shared, or both
- The purpose of each collection (app functionality, analytics, advertising, fraud prevention, etc.)
- Whether data is encrypted in transit
- Whether users can request data deletion
- All third-party SDK data collection (not just first-party)

**Rationale:** Google Play Data Safety is the primary transparency mechanism for Android apps. Inaccurate declarations violate Google Play policy and user trust.

### MASVS-PRIVACY-3.2 — Provide In-App Privacy Disclosure

The app displays a clear privacy notice or consent dialog on first launch (or before first data collection) that explains:
- What data is collected and why
- Whether data is shared with third parties and who
- How long data is retained
- How the user can exercise their data rights

This disclosure is separate from the privacy policy link — it is a user-friendly, in-context summary.

### MASVS-PRIVACY-3.3 — Disclose Non-Obvious Data Collection

Any data collection behavior that a reasonable user would not expect is explicitly disclosed:
- Background data collection (location tracking, sensor data)
- Collection of data from non-primary input sources (clipboard, contacts, calendar)
- Collection that occurs before the user actively engages with the relevant feature
- Data collection by third-party SDKs that the user may not be aware of

### MASVS-PRIVACY-3.4 — Display Privacy Indicators and Notifications

The app respects and complements Android's built-in privacy indicators:
- Does not attempt to circumvent camera/microphone indicators (Android 12+)
- Provides its own in-app indicators when actively using location, camera, or microphone
- Uses `ForegroundServiceType` declarations (`camera`, `microphone`, `location`) to trigger system-level indicators for background usage

**Android References:**
- Android 12+: Green status bar dots for camera/microphone access
- `android:foregroundServiceType="location|camera|microphone"` in manifest

### MASVS-PRIVACY-3.5 — Keep Disclosures Current

Privacy disclosures and the Data Safety Section are updated whenever:
- New data types are collected
- New third-party SDKs are added
- Data sharing practices change
- New features are added that access sensitive data or permissions

A review process ensures that code changes to data collection are reflected in disclosures before release.
