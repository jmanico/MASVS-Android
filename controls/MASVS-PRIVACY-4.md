# MASVS-PRIVACY-4

## Control

The app offers user control over their data.

## Description

Users should have meaningful control over their personal data — the ability to view what is collected, modify their privacy preferences, request data deletion, and revoke consent. On Android, this includes both in-app data management and integration with platform-level privacy controls. This control ensures the app provides these capabilities in compliance with privacy regulations (GDPR, CCPA, LGPD) and platform expectations.

## Android Sub-Requirements

### MASVS-PRIVACY-4.1 — Provide Data Deletion Capability

The app provides a mechanism for users to request deletion of their account and associated data. This mechanism is:
- Accessible from within the app and also from a web URL (required by Google Play policy since December 2023)
- Functional without requiring customer support contact
- Processes deletion requests within a reasonable timeframe
- Confirms when deletion is complete

**Android References:**
- Google Play policy requires a data deletion URL in the Data Safety Section
- Account deletion must delete associated Play Games, subscriptions, and in-app purchases data

### MASVS-PRIVACY-4.2 — Allow Granular Consent Management

The app provides a consent management interface where the user can:
- Grant or deny consent for each data collection purpose separately (analytics, advertising, personalization)
- Modify their consent choices at any time (not just during initial setup)
- Withdraw consent easily (withdrawal must be as easy as granting)
- See which data types are collected under each consent category

### MASVS-PRIVACY-4.3 — Respect Consent Withdrawal

When a user revokes consent:
- The app immediately stops the associated data collection
- Third-party SDKs are reconfigured or disabled for the revoked purpose
- Previously collected data under the revoked consent is either deleted or anonymized, as required by applicable regulation
- The change takes effect without requiring an app restart

### MASVS-PRIVACY-4.4 — Support Data Export / Portability

For apps subject to GDPR or similar regulations, the app provides a mechanism for users to export their personal data in a machine-readable format (JSON, CSV). The export includes all data associated with the user's account.

### MASVS-PRIVACY-4.5 — Re-Prompt on Expanded Data Collection

If the app begins collecting new data types or using existing data for new purposes (e.g., after an update), it re-prompts for consent and updates its transparency disclosures before the new collection begins.

**Rationale:** Initial consent does not cover future, unrelated data uses. Expanding data collection without updated consent violates purpose limitation principles.

### MASVS-PRIVACY-4.6 — Integrate with Android Permission Settings

The app respects system-level privacy controls:
- Honors permission revocation in system Settings — does not repeatedly re-request revoked permissions
- Handles auto-revoked permissions gracefully (unused apps lose permissions on Android 11+)
- Does not use workarounds to circumvent permission denial (e.g., accessing location via WiFi scanning when location permission is denied)

**Android References:**
- Auto-revoke (API 30+): `PackageManager.isAutoRevokeWhitelisted()` to check status
- Permission revocation: respect `ActivityCompat.shouldShowRequestPermissionRationale()` return value
