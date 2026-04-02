# MASVS-PLATFORM-ANDROID: Platform Interaction

Android-specific requirements for MASVS-PLATFORM-1, MASVS-PLATFORM-2, and MASVS-PLATFORM-3.

## MASVS-PLATFORM-1: Secure IPC Mechanisms

### PLATFORM-ANDROID-1.1: Explicit Component Export

All components (Activities, Services, BroadcastReceivers, ContentProviders) MUST explicitly declare `android:exported="true"` or `android:exported="false"` in the manifest. This is mandatory since Android 12 (API 31).

**Testable:** Inspect `AndroidManifest.xml`. Verify all components have explicit `exported` attribute. Build fails if omitted on API 31+.

### PLATFORM-ANDROID-1.2: Minimal Component Exposure

Only components that must be accessible to other apps SHOULD be exported. Sensitive exported components MUST be protected with `signature`-level custom permissions.

**Testable:** List all exported components. Verify each has a documented reason for export. Verify sensitive exported components use signature-level permissions.

### PLATFORM-ANDROID-1.3: Immutable PendingIntents

All `PendingIntent` objects MUST use `FLAG_IMMUTABLE` (default since Android 12) unless mutability is explicitly required for the use case. Mutable PendingIntents MUST NOT be sent to untrusted components.

**Testable:** Static analysis for `PendingIntent.get*()` calls. Verify `FLAG_IMMUTABLE` is set. Verify no mutable PendingIntents exposed to untrusted components.

### PLATFORM-ANDROID-1.4: Intent Redirection Protection

The app MUST NOT forward untrusted Intents to internal components without validation. On Android 16+, the app SHOULD enable Intent Redirection Protection via StrictMode.

**Testable:** Identify Intent forwarding patterns. Verify input validation on forwarded Intents. On Android 16+, verify StrictMode intent redirection detection is enabled.

### PLATFORM-ANDROID-1.5: Package Visibility Scoping

On Android 11+ (API 30), the app MUST use the `<queries>` element in the manifest to declare which packages it needs to interact with, rather than requesting broad package visibility via `QUERY_ALL_PACKAGES`.

**Testable:** Inspect manifest for `<queries>` usage. Verify `QUERY_ALL_PACKAGES` is not declared unless justified and documented.

## MASVS-PLATFORM-2: Secure WebView Usage

### PLATFORM-ANDROID-2.1: Disabled File and Content Access

WebViews MUST disable file and content access:
- `setAllowFileAccess(false)`
- `setAllowContentAccess(false)`
- `setAllowFileAccessFromFileURLs(false)`
- `setAllowUniversalAccessFromFileURLs(false)`

**Testable:** Static analysis verifies WebView settings. Attempt `file://` and `content://` access from WebView. Verify access is denied.

### PLATFORM-ANDROID-2.2: JavaScript Interface Safety

If `addJavascriptInterface` is used, the app MUST target API 17+ (where `@JavascriptInterface` annotation is required). JavaScript interfaces MUST expose only the minimum necessary methods. The interface MUST validate all input from JavaScript.

**Testable:** Verify `@JavascriptInterface` annotation on all exposed methods. Verify minimum API level >= 17. Review exposed methods for input validation.

### PLATFORM-ANDROID-2.3: Safe Browsing Enabled

WebViews SHOULD enable Google Safe Browsing via `setSafeBrowsingEnabled(true)` or manifest meta-data to protect against known malicious URLs.

**Testable:** Verify Safe Browsing is enabled. Navigate to a known test phishing URL. Verify warning is displayed.

### PLATFORM-ANDROID-2.4: URL Allowlisting

WebViews MUST validate loaded URLs against an allowlist. Navigation to unexpected domains MUST be blocked. Consider using `WebViewAssetLoader` for local content.

**Testable:** Attempt to navigate WebView to non-allowlisted URL. Verify navigation is blocked or redirected.

### PLATFORM-ANDROID-2.5: WebView Data Cleanup on Session End

WebView cache, cookies, form data, and local storage MUST be cleared on logout or session termination.

**Testable:** After logout, inspect WebView storage directories. Verify no residual session data.

## MASVS-PLATFORM-3: Secure Platform Feature Usage

### PLATFORM-ANDROID-3.1: Verified Deep Links (App Links)

The app MUST use Android App Links (verified deep links via `assetlinks.json`) instead of unverified custom URL schemes. Custom URL schemes have no ownership verification and are vulnerable to scheme hijacking.

**Testable:** Verify `autoVerify="true"` on intent filters. Verify `.well-known/assetlinks.json` is hosted and valid. Verify custom URL scheme is not used for sensitive actions.

### PLATFORM-ANDROID-3.2: Deep Link Input Validation

All parameters received via deep links MUST be treated as untrusted input and validated/sanitized. Deep links MUST NOT auto-execute sensitive actions (e.g., money transfers, account changes) without additional user confirmation.

**Testable:** Send crafted deep links with malicious parameters. Verify input validation. Verify sensitive actions require user confirmation.

### PLATFORM-ANDROID-3.3: Minimum Permissions at Point of Use

The app MUST request only the minimum necessary runtime permissions, and MUST request them at the point of use with contextual explanation — not at app launch. The app MUST use `shouldShowRequestPermissionRationale()` to provide context.

**Testable:** Launch app fresh. Verify no permission requests at startup. Trigger feature requiring permission. Verify contextual rationale is shown before request.

### PLATFORM-ANDROID-3.4: Selected Photos Access

On Android 14+, the app SHOULD use selected photos access (partial media access) instead of requesting full `READ_MEDIA_IMAGES` / `READ_MEDIA_VIDEO` permissions when the app only needs access to specific user-selected media.

**Testable:** Verify photo picker or selected photos access is used instead of broad media permissions where applicable.

### PLATFORM-ANDROID-3.5: QR Code and NFC Input Validation

Decoded QR code and NFC data MUST be treated as untrusted input. The app MUST display the decoded URL to the user before navigating, validate against an allowlist, and MUST NOT auto-execute actions. NFC payloads for payment or authentication MUST be cryptographically signed.

**Testable:** Scan QR code with malicious URL. Verify URL is displayed for user confirmation. Verify non-allowlisted URLs are blocked or warned.

### PLATFORM-ANDROID-3.6: Tapjacking Protection

Activities displaying sensitive data or accepting sensitive input MUST set `filterTouchesWhenObscured="true"` or check for `FLAG_WINDOW_IS_OBSCURED` to prevent tapjacking (overlay) attacks. Android 16 provides enhanced tapjacking protection.

**Testable:** Attempt tap-through with overlay app on sensitive screen. Verify touches are filtered when obscured.

### PLATFORM-ANDROID-3.7: Manifest Hardening

The `AndroidManifest.xml` MUST be configured with security-hardened defaults:
- `android:debuggable="false"` in release builds
- `android:allowBackup="false"` or scoped `dataExtractionRules`
- Minimal declared permissions
- Minimal exported components

**Testable:** Inspect manifest of release APK. Verify all hardening attributes are correctly set.
