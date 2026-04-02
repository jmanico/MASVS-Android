# MASVS-PLATFORM-1

## Control

The app uses IPC mechanisms securely.

## Description

Android's Inter-Process Communication (IPC) mechanisms — Intents, Content Providers, Broadcast Receivers, and Services — are fundamental to the platform but also a major attack surface. Exported components are accessible to any app on the device. Implicit intents can be intercepted. Content Providers can leak data or be exploited via SQL injection and path traversal. This control ensures all IPC interactions on Android are secured against common attack patterns.

## Android Sub-Requirements

### MASVS-PLATFORM-1.1 — Minimize Exported Components

All components (Activities, Services, Broadcast Receivers, Content Providers) are set to `android:exported="false"` unless they explicitly need to receive intents from other apps. Every `android:exported="true"` component has documented justification and appropriate permission protection.

**Rationale:** Since Android 12 (API 31), components with intent-filters must explicitly declare `android:exported`. Every exported component is an entry point that any app on the device can invoke.

**Android References:**
- `android:exported="false"` in `AndroidManifest.xml` (default for components without intent-filters since API 31)

### MASVS-PLATFORM-1.2 — Protect Exported Components with Permissions

Exported components that handle sensitive operations or data are protected with custom signature-level permissions (`android:protectionLevel="signature"`) or system permissions. The `android:permission` attribute is set on the component declaration.

**Android References:**
```xml
<permission android:name="com.example.SENSITIVE_ACTION"
    android:protectionLevel="signature" />

<activity android:name=".SensitiveActivity"
    android:exported="true"
    android:permission="com.example.SENSITIVE_ACTION" />
```

### MASVS-PLATFORM-1.3 — Use Explicit Intents for Sensitive Communication

The app uses explicit intents (specifying the target component by `ComponentName` or class) for all sensitive IPC. Implicit intents are only used for non-sensitive, user-facing actions (e.g., opening a browser, sharing non-sensitive content).

**Rationale:** Implicit intents can be intercepted by any app that registers a matching intent-filter. On Android 15+, explicit intents must match the target's intent-filter specs, and intents without an action cannot match filters.

### MASVS-PLATFORM-1.4 — Validate All Intent Data

The app validates and sanitizes all data received from intents — extras, data URIs, actions, categories, and clipData. The app does not blindly trust intent data, even from seemingly trusted sources.

**Specific validations:**
- URI scheme, host, and path are validated against an allowlist
- Intent extras are type-checked and range-validated
- The app does not call `Intent.parseUri()` or `Intent.getParcelableExtra()` on untrusted data without validation
- Deep links are validated against expected patterns before navigating

### MASVS-PLATFORM-1.5 — Prevent Intent Redirection

The app does not forward or redirect user-controlled Intent data to `startActivity()`, `startService()`, `sendBroadcast()`, or `setResult()` without validation. Intent redirection allows an attacker to access unexported components through an exported component that acts as a proxy.

**Rationale:** Intent redirection is a critical vulnerability class on Android. An attacker crafts an intent containing a nested intent (in extras), which the vulnerable app then launches on the attacker's behalf, bypassing export restrictions.

**Android References:**
- Android 16+: by-default protection against intent redirection exploits
- `IntentSanitizer` (Jetpack) for validating received intents

### MASVS-PLATFORM-1.6 — Use FLAG_IMMUTABLE for PendingIntents

All `PendingIntent` objects are created with `PendingIntent.FLAG_IMMUTABLE` unless the app requires the intent to be modified by the receiving app (in which case `FLAG_MUTABLE` is used with explicit component, action, and package set on the inner intent).

**Rationale:** Mutable PendingIntents with implicit inner intents can be hijacked — a malicious app can fill in the unspecified fields to redirect the PendingIntent to its own component. `FLAG_IMMUTABLE` is required for apps targeting API 31+.

**Android References:**
- `PendingIntent.getActivity(context, requestCode, intent, FLAG_IMMUTABLE)`
- `PendingIntent.FLAG_IMMUTABLE` (required targeting API 31+)

### MASVS-PLATFORM-1.7 — Secure Content Providers

Content Providers that are exported:
- Use parameterized queries (via `ContentResolver` with selection args) to prevent SQL injection
- Validate file paths in `openFile()` to prevent path traversal (`../../` attacks)
- Set appropriate `android:readPermission` and `android:writePermission`
- Use `android:grantUriPermissions="true"` with `FLAG_GRANT_READ_URI_PERMISSION` for temporary, scoped access instead of broad export
- Use `FileProvider` for file sharing instead of custom Content Provider implementations

**Android References:**
- `FileProvider` — secure file sharing via content URIs
- `ContentProvider.query()` with `selectionArgs` parameter for parameterized queries

### MASVS-PLATFORM-1.8 — Secure Broadcast Receivers

- Broadcast Receivers that handle sensitive data are registered with `ContextCompat.registerReceiver(context, receiver, filter, RECEIVER_NOT_EXPORTED)` (API 33+) or protected with `android:permission`
- The app uses explicit broadcasts (`Intent.setComponent()` or `Intent.setPackage()`) when sending sensitive data
- Ordered broadcasts containing sensitive data specify a permission that receivers must hold
- The app does not send sensitive data via implicit broadcasts

### MASVS-PLATFORM-1.9 — Secure Bound Services

- AIDL-based bound services verify the caller's identity using `Binder.getCallingUid()` and `PackageManager.getPackagesForUid()` before processing sensitive requests
- Services protected with `android:permission` use signature-level permissions for same-developer communication
- The service validates all data received through AIDL/Messenger interfaces

### MASVS-PLATFORM-1.10 — Secure Deep Links and App Links

- Deep link handlers validate the incoming URI scheme, host, path, and query parameters against an allowlist
- Android App Links (verified `https://` links with `android:autoVerify="true"`) are preferred over custom scheme deep links for security-sensitive entry points
- The app does not pass deep link parameters directly to sensitive operations without validation
- The `.well-known/assetlinks.json` file is properly configured on the web server

**Android References:**
- `<intent-filter android:autoVerify="true">` with `ACTION_VIEW` and `https` data scheme
- Digital Asset Links (`.well-known/assetlinks.json`) for domain verification
