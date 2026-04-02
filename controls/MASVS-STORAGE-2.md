# MASVS-STORAGE-2

## Control

The app prevents leakage of sensitive data.

## Description

Sensitive data can be unintentionally exposed through Android platform mechanisms — system logs, clipboard, screenshots, keyboard caches, backups, crash reports, and notification content. These leaks occur as side-effects of using standard APIs and system features. This control ensures developers actively prevent these unintentional data exposures using Android-specific mitigations.

## Android Sub-Requirements

### MASVS-STORAGE-2.1 — No Sensitive Data in System Logs

The app does not write sensitive data (credentials, tokens, PII, session IDs) to system logs via `android.util.Log` or `System.out`/`System.err`. Release builds strip verbose and debug log statements using R8/ProGuard rules.

**Rationale:** On Android versions prior to 4.1, any app could read the system log. On modern Android, logs are accessible via ADB, crash reporting services, and on rooted devices. Log output from production apps should never contain secrets.

**Android References:**
- R8 rule: `-assumenosideeffects class android.util.Log { public static int v(...); public static int d(...); }`
- `Timber` library with a no-op `Tree` for release builds

### MASVS-STORAGE-2.2 — Prevent Screenshots and Screen Recording of Sensitive Screens

Activities displaying sensitive information (passwords, financial data, OTPs) set `WindowManager.LayoutParams.FLAG_SECURE` to prevent screenshots, screen recording, and display on non-secure outputs (Chromecast, screen mirroring).

**Rationale:** Android captures screenshots for the Recent Apps screen. Without `FLAG_SECURE`, sensitive data appears in the recents thumbnail and can be captured by screen recording apps or remote display.

**Android References:**
- `getWindow().setFlags(FLAG_SECURE, FLAG_SECURE)` in `Activity.onCreate()`
- Android 15+: `WindowManager.addScreenRecordingCallback()` for detection (complementary, not a replacement for FLAG_SECURE)

### MASVS-STORAGE-2.3 — Disable Keyboard Cache for Sensitive Input Fields

Text input fields accepting sensitive data (passwords, credit card numbers, SSNs) use `android:inputType="textNoSuggestions|textPassword"` or `InputType.TYPE_TEXT_FLAG_NO_SUGGESTIONS` to disable keyboard autocomplete, suggestions, and caching.

**Rationale:** The Android keyboard (IME) caches typed text for autocomplete suggestions. This cache persists on disk and can expose previously typed sensitive values.

**Android References:**
- `android:inputType="textPassword"` — disables suggestions and shows dots
- `android:inputType="textNoSuggestions"` — disables suggestions while showing text
- `EditText.setInputType(InputType.TYPE_TEXT_FLAG_NO_SUGGESTIONS | InputType.TYPE_TEXT_VARIATION_PASSWORD)`

### MASVS-STORAGE-2.4 — Prevent Sensitive Data in Clipboard

The app prevents copying of sensitive data to the clipboard. Where copy functionality is unavoidable, the app uses `ClipboardManager.setPrimaryClip()` with `ClipDescription.EXTRA_IS_SENSITIVE` (API 33+) to mark content as sensitive, preventing it from appearing in clipboard previews.

**Rationale:** On Android 12 and below, any foreground app can read the clipboard. Android 12+ shows a toast notification on clipboard access. Android 13+ supports auto-clearing and sensitive content flags, but the data is still briefly accessible.

**Android References:**
- `android:longClickable="false"` or custom `ActionMode.Callback` to disable copy on `TextView`/`EditText`
- `ClipDescription.EXTRA_IS_SENSITIVE` (API 33+) — marks clipboard content as sensitive
- `ClipData.newPlainText()` with flag for transient data

### MASVS-STORAGE-2.5 — Mask Sensitive Data in Notifications

Notifications do not contain sensitive data (OTP codes, account balances, message previews with PII) in their content or title. Where sensitive data is necessary, the app uses `Notification.VISIBILITY_SECRET` or `Notification.VISIBILITY_PRIVATE` with a redacted public version.

**Rationale:** Notifications are visible on the lock screen, in the notification shade, on connected wearables, and in notification history. Sensitive content in notifications is exposed without device unlock.

**Android References:**
- `NotificationCompat.Builder.setVisibility(VISIBILITY_PRIVATE)`
- `NotificationCompat.Builder.setPublicVersion(redactedNotification)`
- `NotificationChannel.setLockscreenVisibility(VISIBILITY_SECRET)`

### MASVS-STORAGE-2.6 — Disable Autofill for Sensitive Fields Where Inappropriate

Input fields that should not be autofilled (OTP fields, security questions, temporary codes) set `android:importantForAutofill="no"` to opt out of the Android Autofill Framework.

**Rationale:** Autofill services are third-party apps with access to field contents. While legitimate password managers are helpful, autofill of temporary or security-sensitive values introduces unnecessary exposure to the autofill provider.

**Android References:**
- `android:importantForAutofill="no"` — excludes the view from autofill
- `android:importantForAutofill="noExcludeDescendants"` — excludes the view and all children

### MASVS-STORAGE-2.7 — Prevent Data Leakage via App Backgrounding

The app clears or obscures sensitive UI content when entering the background (`onPause()`/`onStop()`) to prevent exposure in the Recent Apps screen thumbnail.

**Rationale:** Even with `FLAG_SECURE`, some OEMs capture app thumbnails differently. Proactively clearing sensitive views provides defense-in-depth.

### MASVS-STORAGE-2.8 — Exclude Sensitive Data from Crash Reports and Analytics

The app's crash reporting and analytics SDKs are configured to exclude sensitive data. Custom keys, breadcrumbs, and user attributes do not contain PII, credentials, or session tokens.

**Rationale:** Crash reports are stored on third-party servers (Firebase Crashlytics, Sentry, etc.) and accessible to development teams. Sensitive data in crash reports creates a secondary data exposure surface.

### MASVS-STORAGE-2.9 — Prevent WebView Cache and Storage Leaks

WebViews that display sensitive content are configured to disable caching (`WebSettings.setCacheMode(LOAD_NO_CACHE)`), clear the cache on navigation away, and disable DOM storage / database storage for sensitive pages.

**Rationale:** WebView caches (HTTP cache, DOM storage, Web SQL databases) persist on the filesystem and can expose sensitive data rendered in web content.

**Android References:**
- `WebSettings.setCacheMode(WebSettings.LOAD_NO_CACHE)`
- `WebView.clearCache(true)` — clears disk and memory cache
- `WebSettings.setDomStorageEnabled(false)` for sensitive contexts
