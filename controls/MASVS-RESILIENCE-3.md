# MASVS-RESILIENCE-3

## Control

The app implements anti-static analysis mechanisms.

## Description

Understanding the internals of an Android app through static analysis (decompiling DEX bytecode with jadx, reverse engineering native libraries with Ghidra/IDA) is typically the first step toward tampering. This control ensures the app makes static analysis significantly more difficult through obfuscation, string encryption, and code hardening techniques.

## Android Sub-Requirements

### MASVS-RESILIENCE-3.1 — Enable R8 Code Shrinking and Obfuscation

R8 is enabled for all release builds with `minifyEnabled true` and `shrinkResources true`. The ProGuard/R8 configuration preserves only essential keep rules and does not over-preserve classes.

**Android References:**
```groovy
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                'proguard-rules.pro'
        }
    }
}
```

**Rationale:** R8 renames classes, methods, and fields to short, meaningless names and removes unused code. While not strong obfuscation, it is the baseline that all Android apps should use.

### MASVS-RESILIENCE-3.2 — Encrypt String Literals Containing Sensitive Values

Sensitive strings (API endpoint URLs, encryption algorithm names, key aliases, error messages that reveal logic) are not present as plaintext in the DEX file. The app uses string encryption (via commercial obfuscation tools or build-time string transformation) to prevent extraction via `strings` or `jadx`.

**Rationale:** R8 does NOT encrypt strings. All string literals remain in plaintext in the DEX file and are trivially extractable. API keys, endpoint URLs, and logic-revealing strings are immediately visible to any reverse engineer.

### MASVS-RESILIENCE-3.3 — Apply Control Flow Obfuscation (High-Value Apps)

For high-value apps (financial, DRM, gaming with anti-cheat), the app uses commercial obfuscation tools that implement control flow obfuscation (opaque predicates, control flow flattening, bogus code insertion) beyond what R8 provides.

**Rationale:** R8 only performs identifier renaming and dead code removal. The algorithm logic, control flow, and program structure remain clearly readable in decompiled output. Control flow obfuscation makes algorithmic understanding significantly harder.

**Commercial tools:**
- Guardsquare DexGuard — Android-specific, includes string encryption, class encryption, asset encryption
- Promon SHIELD — runtime protection + obfuscation
- Verimatrix — code hardening and white-box cryptography

### MASVS-RESILIENCE-3.4 — Obfuscate Native Code

If the app contains native libraries (`.so` files), native code is obfuscated using LLVM-based obfuscators (O-LLVM, Hikari, Arkari) with:
- Control flow flattening
- Instruction substitution
- String encryption
- Symbol stripping (`-fvisibility=hidden`, `strip`)

**Rationale:** Native code is decompilable with Ghidra or IDA Pro. Default compilation produces readable disassembly with clear control flow and recognizable function names.

### MASVS-RESILIENCE-3.5 — Remove Debug Information from Release Builds

Release builds are configured with:
- `android:debuggable="false"` in manifest (default for release builds)
- Debug symbols stripped from native libraries
- Log statements removed via R8 rules: `-assumenosideeffects class android.util.Log { *; }`
- No `BuildConfig.DEBUG` code paths active in release

**Rationale:** Debug information (line numbers, local variable names, debug symbols) makes reverse engineering significantly easier. Log statements may reveal internal state and logic.
