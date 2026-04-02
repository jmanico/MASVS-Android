# MASVS-CODE: Code Quality and Supply Chain

## Overview

Android apps are distributed as APK/AAB bundles containing DEX bytecode, native libraries, resources, and third-party dependencies fetched via Gradle from Maven repositories. This distribution model creates attack surface at multiple levels — outdated platform dependencies, unpatched third-party libraries, malicious supply chain injections, and insufficient input validation. This category ensures the app maintains code quality, manages its dependency supply chain securely, and validates all untrusted inputs.

## Android Build and Distribution Security

| Concern | Android Mechanism |
|---|---|
| Platform version | `minSdkVersion` / `targetSdkVersion` in `build.gradle` |
| Forced updates | Google Play In-App Updates API (`com.google.android.play:app-update`) |
| Dependency scanning | OWASP Dependency-Check, Snyk, GitHub Dependabot |
| Dependency integrity | `gradle/verification-metadata.xml` (SHA-256 + PGP verification) |
| Dependency locking | `dependenciesLocking { lockAllConfigurations() }` + `gradle.lockfile` |
| SBOM generation | CycloneDX Gradle plugin |
| App signing | APK Signature Scheme v2/v3/v4 via `apksigner` |
| Code shrinking | R8 (`minifyEnabled true`) |
| Resource shrinking | R8 (`shrinkResources true`) |

## Input Validation Attack Surface

| Input Source | Attack | Mitigation |
|---|---|---|
| Intent extras/data | Type confusion, deserialization | Type-safe `getParcelableExtra()` with class param (API 33+) |
| Content Provider queries | SQL injection | Parameterized queries with `selectionArgs` |
| `ContentProvider.openFile()` | Path traversal | Canonical path validation |
| ZIP archives | ZipSlip (path traversal) | Validate `ZipEntry` names before extraction |
| WebView content | XSS, JavaScript injection | HTML sanitization before `loadData()` |
| Deep links | Parameter injection | URI scheme/host/param validation |
| NFC/Bluetooth data | Malformed messages | Format and range validation |

## Controls

| Control | Sub-Requirements | Focus |
|---|---|---|
| [MASVS-CODE-1](../controls/MASVS-CODE-1.md) | 4 sub-requirements | Platform version requirements |
| [MASVS-CODE-2](../controls/MASVS-CODE-2.md) | 4 sub-requirements | Enforced app updates |
| [MASVS-CODE-3](../controls/MASVS-CODE-3.md) | 5 sub-requirements | Dependency and supply chain security |
| [MASVS-CODE-4](../controls/MASVS-CODE-4.md) | 8 sub-requirements | Input validation and sanitization |

## OWASP Mobile Top 10 2024 Mapping

- **M2 — Inadequate Supply Chain Security:** Directly addressed by MASVS-CODE-3
- **M4 — Insufficient Input/Output Validation:** Directly addressed by MASVS-CODE-4
- **M7 — Insufficient Binary Protections:** Partially addressed (see MASVS-RESILIENCE for full coverage)
