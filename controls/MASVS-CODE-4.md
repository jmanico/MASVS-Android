# MASVS-CODE-4

## Control

The app validates and sanitizes all untrusted inputs.

## Description

Android apps receive input from many sources: user interface fields, intents from other apps, deep links, content providers, the network, local files, clipboard, and NFC. All of these inputs can be crafted by an attacker. This control ensures the app treats all external input as untrusted and applies appropriate validation and sanitization to prevent injection attacks and logic bypasses on Android.

## Android Sub-Requirements

### MASVS-CODE-4.1 — Validate Intent Extras and Data URIs

All data extracted from incoming intents — extras (`getStringExtra()`, `getParcelableExtra()`), data URIs (`getData()`), actions, and categories — is validated against expected types, ranges, and patterns before use.

**Specific risks:**
- `getParcelableExtra()` can trigger deserialization of attacker-controlled classes
- `getData()` URIs may contain unexpected schemes (`javascript:`, `file:`, `content:`)
- Missing extras should be handled gracefully (null checks), not cause crashes

**Android References:**
- Use `IntentCompat.getParcelableExtra()` with explicit class parameter (API 33+)
- Validate URI schemes against an allowlist before calling `startActivity()` or loading in WebView

### MASVS-CODE-4.2 — Prevent SQL Injection in Content Providers

Content Provider `query()`, `update()`, `delete()` operations use parameterized queries with `selectionArgs` for all user-controlled values. Selection strings are never built via string concatenation with untrusted input.

**Secure pattern:**
```java
// CORRECT: parameterized query
cursor = resolver.query(uri, projection, "name = ?", new String[]{userInput}, null);

// INSECURE: string concatenation
cursor = resolver.query(uri, projection, "name = '" + userInput + "'", null, null);
```

### MASVS-CODE-4.3 — Prevent Path Traversal in File Operations

The app validates file paths received from intents, content URIs, or user input to prevent directory traversal attacks. When implementing `ContentProvider.openFile()`, the resolved path is validated to be within the expected base directory.

**Secure pattern:**
```java
@Override
public ParcelFile openFile(Uri uri, String mode) {
    File file = new File(getContext().getFilesDir(), uri.getLastPathSegment());
    // Validate canonical path is within base directory
    if (!file.getCanonicalPath().startsWith(getContext().getFilesDir().getCanonicalPath())) {
        throw new SecurityException("Path traversal detected");
    }
    return ParcelFileDescriptor.open(file, ParcelFileDescriptor.MODE_READ_ONLY);
}
```

### MASVS-CODE-4.4 — Prevent Zip Path Traversal (ZipSlip)

When extracting ZIP archives, the app validates that each `ZipEntry` name does not contain path traversal sequences (`../`). The resolved extraction path is verified to be within the target directory.

**Rationale:** Zip path traversal (ZipSlip) allows an attacker to overwrite arbitrary files on the filesystem by crafting ZIP entries with `../../` in their names. This can overwrite native libraries, DEX files, or configuration files.

### MASVS-CODE-4.5 — Sanitize Data Before WebView Loading

Data loaded into WebViews via `loadUrl()`, `loadData()`, `loadDataWithBaseURL()`, or `evaluateJavascript()` is sanitized for HTML/JavaScript injection. User-controlled data is never interpolated directly into HTML or JavaScript strings.

**Rationale:** Unsanitized data in WebView content enables XSS within the app's WebView context. If a JavaScript interface bridge is present, XSS escalates to native code execution.

### MASVS-CODE-4.6 — Validate Deep Link Parameters

Parameters received via deep links (`https://` App Links or custom scheme URIs) are validated against expected patterns, types, and ranges. Deep link parameters are never used directly in sensitive operations (database queries, file access, navigation to arbitrary activities) without validation.

**Rationale:** Deep links are externally accessible — any app or website can invoke them. All deep link parameters should be treated with the same suspicion as network input.

### MASVS-CODE-4.7 — Handle Deserialization Safely

The app avoids Java serialization (`Serializable`) for untrusted data. If `Parcelable` objects are received from untrusted sources, the app uses explicit class parameters for `getParcelableExtra()` and `getParcelableArrayListExtra()` (API 33+) to prevent deserialization of unexpected classes.

**Rationale:** Java deserialization can trigger arbitrary code execution via gadget chains. Android's `Parcelable` is safer but still requires type verification when receiving from untrusted sources.

### MASVS-CODE-4.8 — Validate NFC and Bluetooth Input

If the app processes NFC (NDEF messages) or Bluetooth data, all received data is validated against expected message types and formats. NFC tags and Bluetooth peers are treated as untrusted input sources.

**Rationale:** NFC tags can be written by anyone and placed in public locations. Bluetooth data can be spoofed. Both are physical-proximity attack vectors.
