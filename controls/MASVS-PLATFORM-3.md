# MASVS-PLATFORM-3

## Control

The app uses the user interface securely.

## Description

Sensitive data displayed in the Android UI — passwords, credit card numbers, OTP codes, financial balances — can be exposed through platform mechanisms such as screenshots for the Recent Apps screen, screen overlays (tapjacking), shoulder surfing, screen sharing, and accessibility services. This control ensures the app's UI properly protects displayed sensitive information using Android-specific mitigations.

## Android Sub-Requirements

### MASVS-PLATFORM-3.1 — Prevent Tapjacking / Overlay Attacks

Views that perform security-sensitive actions (login, payment authorization, permission grants) are protected against overlay attacks:
- Set `android:filterTouchesWhenObscured="true"` on sensitive views in layout XML
- Override `onFilterTouchEventForSecurity()` to check `MotionEvent.FLAG_WINDOW_IS_OBSCURED` and `FLAG_WINDOW_IS_PARTIALLY_OBSCURED`
- Reject touch events when the view is obscured by another window

**Rationale:** Tapjacking (overlay attacks) involve a malicious app drawing a transparent or misleading overlay on top of the legitimate app's UI, tricking the user into tapping sensitive controls. Research in 2025 ("TapTrap") demonstrated animation-driven tapjacking on Android 15+ without overlay permission.

**Android References:**
- `android:filterTouchesWhenObscured="true"` in layout XML
- `View.onFilterTouchEventForSecurity(MotionEvent)` — return false to reject obscured touches
- `MotionEvent.FLAG_WINDOW_IS_OBSCURED` and `FLAG_WINDOW_IS_PARTIALLY_OBSCURED`

### MASVS-PLATFORM-3.2 — Use FLAG_SECURE for Sensitive Screens

Activities displaying sensitive information set `FLAG_SECURE` to prevent:
- Screenshots and screen recording
- Display on non-secure outputs (casting, mirroring)
- Thumbnail capture for the Recent Apps screen

(Cross-reference: MASVS-STORAGE-2.2)

**Android References:**
- `getWindow().setFlags(WindowManager.LayoutParams.FLAG_SECURE, WindowManager.LayoutParams.FLAG_SECURE)`

### MASVS-PLATFORM-3.3 — Detect Screen Recording (Android 15+)

On Android 15+, apps handling sensitive data register a `WindowManager.addScreenRecordingCallback()` to detect when the screen is being recorded and take appropriate action (obscuring sensitive content, pausing sensitive operations, or notifying the user).

**Rationale:** `FLAG_SECURE` prevents screenshots and recording. The screen recording callback provides additional awareness — the app can adjust behavior (e.g., hide account balances) when recording is active, even for non-FLAG_SECURE screens.

### MASVS-PLATFORM-3.4 — Mask Sensitive Data in the UI

Sensitive data displayed in the UI is masked or partially redacted by default:
- Passwords use `android:inputType="textPassword"` (shows dots)
- Credit card numbers show only the last 4 digits
- Account numbers and SSNs are partially masked
- A "reveal" toggle is provided where the user needs to see the full value

### MASVS-PLATFORM-3.5 — Prevent Task Hijacking

The app mitigates StrandHogg/task hijacking attacks:
- Set `android:taskAffinity=""` (empty string) on sensitive Activities to prevent task affinity manipulation
- Set `android:launchMode="singleTask"` or `android:launchMode="singleInstance"` for the main Activity
- Consider `android:allowTaskReparenting="false"` to prevent Activities from being moved between tasks

**Rationale:** Task hijacking (StrandHogg) allows a malicious app to place its Activity on top of the legitimate app's task stack by manipulating task affinity. The user believes they are interacting with the legitimate app but is actually providing credentials to the malicious app.

### MASVS-PLATFORM-3.6 — Secure Accessibility Service Interaction

The app does not expose sensitive data unnecessarily to accessibility services:
- Sensitive views set `android:importantForAccessibility="no"` where accessibility is not needed
- Password fields use `android:inputType="textPassword"` which automatically masks content from accessibility services
- The app does not broadcast sensitive data through `AccessibilityEvent` custom content descriptions

**Rationale:** Accessibility services have broad access to UI content and events. While legitimate accessibility services are essential, malicious apps can abuse accessibility permissions to read all displayed text, capture credentials, and perform actions on behalf of the user.
