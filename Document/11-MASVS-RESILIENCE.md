# MASVS-RESILIENCE: Resilience Against Reverse Engineering and Tampering

## Overview

Android APKs are trivially decompilable — `jadx` produces readable Java/Kotlin from DEX bytecode, `apktool` extracts resources and manifest, and tools like Frida and Xposed allow runtime manipulation of any app function. For apps that handle financial transactions, DRM, anti-cheat, or sensitive business logic, defense-in-depth measures are necessary to increase the cost and complexity of reverse engineering and tampering. This category covers platform integrity validation, anti-tampering, anti-static analysis, and anti-dynamic analysis techniques on Android.

**Important:** MASVS-RESILIENCE controls are defense-in-depth. They raise the bar but do not provide absolute protection. Critical security logic must always be enforced server-side. These controls complement, not replace, the other MASVS categories.

## Android Resilience Landscape

| Layer | Threats | Defenses |
|---|---|---|
| Platform integrity | Root (Magisk/KernelSU), custom ROM, unlocked bootloader | Play Integrity API, multi-signal root detection, emulator detection |
| App integrity | APK repackaging, DEX patching, resource modification | Signing verification, Play Integrity app verdict, checksum verification |
| Static analysis | jadx decompilation, strings extraction, manifest analysis | R8 obfuscation, string encryption, control flow obfuscation, native obfuscation |
| Dynamic analysis | Frida hooking, Xposed modules, debugger attachment | Anti-Frida, anti-debug, anti-hook, timing checks |
| Runtime environment | Screen capture, accessibility abuse, overlay attacks | Play Integrity app access risk, FLAG_SECURE, tapjacking protection |

## Commercial vs. Open-Source Tools

| Tool | Type | Capabilities |
|---|---|---|
| R8 (Google) | Free | Identifier renaming, dead code removal, optimization |
| Guardsquare DexGuard | Commercial | String encryption, class encryption, control flow obfuscation, RASP |
| Promon SHIELD | Commercial | Runtime protection, integrity checks, anti-debugging, anti-hooking |
| Verimatrix | Commercial | Code hardening, white-box cryptography, app shielding |
| O-LLVM / Hikari | Open-source | Native code obfuscation (LLVM-based) |
| RootBeer | Open-source | Root detection library (community-maintained) |

## Controls

| Control | Sub-Requirements | Focus |
|---|---|---|
| [MASVS-RESILIENCE-1](../controls/MASVS-RESILIENCE-1.md) | 5 sub-requirements | Platform integrity validation (root/emulator detection) |
| [MASVS-RESILIENCE-2](../controls/MASVS-RESILIENCE-2.md) | 6 sub-requirements | Anti-tampering (signing, integrity, repackaging detection) |
| [MASVS-RESILIENCE-3](../controls/MASVS-RESILIENCE-3.md) | 5 sub-requirements | Anti-static analysis (obfuscation, string encryption) |
| [MASVS-RESILIENCE-4](../controls/MASVS-RESILIENCE-4.md) | 6 sub-requirements | Anti-dynamic analysis (anti-debug, anti-Frida, anti-hook) |

## OWASP Mobile Top 10 2024 Mapping

- **M7 — Insufficient Binary Protections:** Directly addressed by all four controls
