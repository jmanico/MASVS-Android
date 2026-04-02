# MASVS-RESILIENCE-4

## Control

The app implements anti-dynamic analysis techniques.

## Description

Dynamic analysis — attaching debuggers, hooking functions with Frida, instrumenting with Xposed/LSPosed, and runtime manipulation — allows attackers to modify app behavior, bypass security checks, and extract secrets at runtime. This control ensures the app detects and resists dynamic instrumentation and debugging, making runtime tampering significantly harder.

## Android Sub-Requirements

### MASVS-RESILIENCE-4.1 — Detect Debugging

The app detects when a debugger is attached:
- Check `android.os.Debug.isDebuggerConnected()` — Java debugger detection
- Check `TracerPid` in `/proc/self/status` — native debugger (ptrace) detection
- Use `ptrace(PTRACE_TRACEME, 0, 0, 0)` in native code — self-trace to prevent debugger attachment
- Monitor timing anomalies that indicate single-stepping

**Rationale:** Debuggers allow an attacker to inspect memory, modify variables, set breakpoints on security checks, and step through cryptographic operations.

### MASVS-RESILIENCE-4.2 — Detect Frida and Dynamic Instrumentation Frameworks

The app detects Frida, Xposed, and other instrumentation frameworks:

| Framework | Detection Technique |
|---|---|
| Frida (server) | Check for Frida default port 27042; scan `/proc/self/maps` for `frida-agent`, `frida-gadget` |
| Frida (gadget) | Check for Frida-specific named threads (`gmain`, `gdbus`, `gum-js-loop`) in `/proc/self/task/*/comm` |
| Frida (hooks) | Verify integrity of function prologues in libc (compare first bytes of `open`, `read`, `write` against known-good values) |
| Xposed/LSPosed | Check for Xposed Installer/LSPosed Manager packages; scan `/proc/self/maps` for Xposed bridge libraries |
| Magisk (Zygisk) | Check for Zygisk modules in `/data/adb/modules/`; detect injected libraries in `/proc/self/maps` |

**Rationale:** Frida is the most widely used dynamic instrumentation tool for Android reverse engineering. It can hook any Java or native function, modify arguments and return values, and bypass virtually any client-side security check.

### MASVS-RESILIENCE-4.3 — Implement Anti-Hook Techniques

The app verifies the integrity of critical security functions:
- Compare function prologue bytes of critical native functions against known-good values (detects inline hooks)
- Use `dlsym()` to resolve function addresses and verify they point to expected library regions
- Implement critical checks using raw syscalls (`syscall()`) instead of libc wrappers that can be hooked
- Use JNI `RegisterNatives()` for dynamic native method registration (harder to hook than default JNI naming)

**Rationale:** Frida and similar tools work by replacing function prologues with trampolines to hook code. Verifying function integrity detects these modifications.

### MASVS-RESILIENCE-4.4 — Detect Runtime Environment Manipulation

The app detects signs of runtime manipulation:
- Check `Settings.Global.ADB_ENABLED` — ADB connection active
- Verify `ApplicationInfo.FLAG_DEBUGGABLE` is not set in the running process
- Detect suspicious environment variables (`CLASSPATH` modifications for Xposed)
- Monitor for unexpected loaded libraries in `/proc/self/maps`

### MASVS-RESILIENCE-4.5 — Implement Timing-Based Detection

The app uses timing checks to detect single-stepping and breakpoint-based debugging:
- Measure execution time of critical code sections
- Compare against expected baselines
- Flag significant timing anomalies that indicate debugger intervention

**Rationale:** Debuggers and instrumentation tools introduce latency. A code section that normally executes in microseconds taking seconds indicates single-stepping or breakpoint activity.

### MASVS-RESILIENCE-4.6 — Use App Access Risk Verdict (Android 15+)

For apps handling sensitive data, leverage Play Integrity API's `appAccessRiskVerdict` to detect:
- `UNKNOWN_CAPTURING` — untrusted apps are screen-capturing
- `UNKNOWN_CONTROLLING` — untrusted apps have control access (accessibility, remote control)
- `UNKNOWN_OVERLAYS` — untrusted apps are drawing overlays

**Rationale:** Screen capture and accessibility-based control apps can exfiltrate displayed data and automate actions on behalf of the user. The App Access Risk verdict detects these runtime risks.

**Android References:**
- Play Integrity API response field: `environmentDetails.appAccessRiskVerdict`
- Available for apps distributed via Google Play with `MEETS_DEVICE_INTEGRITY`
