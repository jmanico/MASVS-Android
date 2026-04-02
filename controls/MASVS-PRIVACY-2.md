# MASVS-PRIVACY-2

## Control

The app prevents identification of the user.

## Description

Protecting user identity and preventing cross-app or cross-session tracking is a core privacy requirement. Android provides various device identifiers and fingerprinting signals that can be misused for tracking. This control ensures the app uses privacy-preserving identifiers, implements anonymization and pseudonymization techniques, and prevents identifier misuse.

## Android Sub-Requirements

### MASVS-PRIVACY-2.1 — Do Not Use Hardware Device Identifiers

The app does not access or transmit hardware device identifiers for user tracking:
- `IMEI` (removed from non-privileged access since API 29)
- `MEID`
- Device serial number (`Build.SERIAL`, restricted since API 29)
- MAC addresses (randomized since API 29; `WifiInfo.getMacAddress()` returns `02:00:00:00:00:00`)
- `ANDROID_ID` (scoped per-app per-user since API 26, but still a long-lived identifier)

**Rationale:** Hardware identifiers are permanent, non-resettable, and enable cross-app tracking and device fingerprinting.

### MASVS-PRIVACY-2.2 — Use Privacy-Preserving Identifiers

The app uses the appropriate identifier for each use case:
- **Advertising:** Google Advertising ID (`AdvertisingIdClient.getId()`) with user-resettable opt-out (or Privacy Sandbox Topics API)
- **Analytics:** App-instance ID that resets on reinstall (e.g., `FirebaseInstallations.getId()`)
- **Push notifications:** FCM registration token (scoped to app instance)
- **Fraud detection:** Server-generated opaque tokens, not device fingerprints

### MASVS-PRIVACY-2.3 — Prevent Cross-Purpose Identifier Linking

Identifiers collected for one purpose (e.g., fraud detection) are not reused for another purpose (e.g., analytics, advertising). The app enforces isolation between identifier streams.

**Rationale:** Cross-purpose linking enables building comprehensive user profiles from individually limited data streams. A fraud-detection device fingerprint repurposed for ad targeting violates the principle of purpose limitation.

### MASVS-PRIVACY-2.4 — Implement Data Anonymization for Analytics

Analytics data transmitted to third-party services is anonymized:
- IP addresses are truncated or not collected (e.g., Google Analytics IP anonymization)
- User IDs are pseudonymized (hashed, not raw)
- Location data is generalized (city-level, not precise coordinates)
- Event data is aggregated where possible

### MASVS-PRIVACY-2.5 — Minimize Device Fingerprinting Signals

The app does not collect or transmit combinations of device attributes that could be used for fingerprinting:
- Screen resolution + OS version + installed fonts + timezone + language
- Sensor data (accelerometer calibration, battery characteristics)
- Installed app lists (restricted via package visibility since API 30)
- WebView `navigator.userAgent` strings (consider overriding with generic values)

**Android References:**
- Android 16: `MediaStore` version strings are now app-specific to prevent fingerprinting
- Package visibility restrictions (API 30+): `<queries>` required to see other apps
