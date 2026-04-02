# MASVS-PLATFORM-2

## Control

The app uses WebViews securely.

## Description

Android WebViews (`android.webkit.WebView`) embed web content within native apps and bridge JavaScript to native code. Misconfigured WebViews are a major attack surface — they can expose native functionality to malicious web content, leak sensitive data through caches and cookies, and enable cross-site scripting within the app's context. This control ensures WebViews are configured with the principle of least privilege and that JavaScript bridges are properly secured.

## Android Sub-Requirements

### MASVS-PLATFORM-2.1 — Disable JavaScript Unless Required

WebViews have JavaScript disabled by default (`WebSettings.setJavaScriptEnabled(false)`). JavaScript is only enabled when the WebView must render interactive web content that requires it, and only for trusted content.

**Rationale:** JavaScript enables dynamic execution of code within the WebView. If the WebView loads untrusted content (user-generated, third-party, or attacker-controlled), JavaScript can be used for XSS, data exfiltration, and native bridge exploitation.

### MASVS-PLATFORM-2.2 — Secure JavaScript Interface Bridges

If `addJavascriptInterface()` is used:
- The interface object exposes only the minimum necessary methods
- All methods annotated with `@JavascriptInterface` validate their inputs
- The interface does not expose access to sensitive data, file system operations, or native capabilities that could be abused
- The WebView only loads content from trusted, verified origins

**Rationale:** `addJavascriptInterface()` exposes native Java/Kotlin methods to JavaScript running in the WebView. On API 16 and below, all public methods (including inherited ones from `Object`) were accessible — a critical RCE vulnerability. On API 17+, only `@JavascriptInterface`-annotated methods are exposed, but they remain callable by any JavaScript running in the WebView.

### MASVS-PLATFORM-2.3 — Disable File Access in WebViews

WebViews are configured with:
- `WebSettings.setAllowFileAccess(false)` — disables `file://` scheme access
- `WebSettings.setAllowFileAccessFromFileURLs(false)` — prevents file URLs from accessing other file URLs
- `WebSettings.setAllowUniversalAccessFromFileURLs(false)` — prevents file URLs from accessing any origin
- `WebSettings.setAllowContentAccess(false)` — disables `content://` scheme access (unless specifically needed)

**Rationale:** File access from WebViews enables local file theft. A malicious page loaded in the WebView can read files from the app's internal storage (credentials, databases) if file access is enabled.

### MASVS-PLATFORM-2.4 — Validate URLs in shouldOverrideUrlLoading

The app implements `WebViewClient.shouldOverrideUrlLoading()` and validates all navigation URLs against an allowlist of trusted schemes and domains. Navigation to `javascript:`, `data:`, `file:`, and untrusted `https://` domains is blocked.

**Rationale:** Without URL validation, an attacker can navigate the WebView to malicious content via crafted links, redirects, or JavaScript. Scheme validation prevents `javascript:` URI exploitation and `file://` access.

### MASVS-PLATFORM-2.5 — Handle SSL Errors Correctly in WebViews

The app's `WebViewClient.onReceivedSslError()` implementation calls `handler.cancel()` for all SSL errors. It never calls `handler.proceed()` in production builds. The user may be shown an error message, but the connection is always terminated.

**Rationale:** Calling `handler.proceed()` on SSL errors disables certificate validation for the WebView, enabling MITM attacks on all content loaded in that WebView. Google Play rejects apps that call `handler.proceed()`.

### MASVS-PLATFORM-2.6 — Clear WebView Data After Sensitive Sessions

After loading sensitive content, the app clears WebView state:
- `WebView.clearCache(true)` — clears disk and memory cache
- `WebView.clearHistory()` — clears navigation history
- `CookieManager.getInstance().removeAllCookies(null)` — clears cookies
- `WebStorage.getInstance().deleteAllData()` — clears Web Storage

**Rationale:** WebView caches, cookies, and local storage persist on the filesystem and can be extracted from the device.

### MASVS-PLATFORM-2.7 — Use WebView Safe Browsing

The app enables Safe Browsing for WebViews via manifest metadata to protect users from known malicious URLs:
```xml
<meta-data android:name="android.webkit.WebView.EnableSafeBrowsing"
    android:value="true" />
```

**Android References:**
- Available since API 26; enabled by default since API 28 but can be explicitly declared for clarity
- `WebView.startSafeBrowsing(context, callback)` for async initialization
