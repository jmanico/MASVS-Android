# OWASP MASVS for Android

[![OWASP Incubator](https://img.shields.io/badge/owasp-incubator-blue.svg)](https://owasp.org/www-project-mobile-app-security/)
[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

**An Android-specific adaptation of the [OWASP Mobile Application Security Verification Standard (MASVS)](https://github.com/OWASP/owasp-masvs)** with detailed, platform-native requirements, API-level guidance, and implementation references.

## What Is This?

The upstream OWASP MASVS is intentionally platform-agnostic — it defines *what* to verify, not *how* on a specific OS. This project takes each MASVS control and expands it into concrete, testable Android sub-requirements that reference specific APIs, manifest attributes, configuration files, and platform behaviors.

**This is NOT an official OWASP project.** It is a community fork maintained by [Jim Manico](https://github.com/jmanico) for teams that need Android-specific security requirements they can hand directly to developers, auditors, and pentesters.

## Target Audience

| Role | How to Use This |
|---|---|
| **Android Developers** | Implement each sub-requirement using the referenced APIs and configurations |
| **Security Auditors** | Use sub-requirements as a checklist; each is testable and maps back to upstream MASVS |
| **Pentesters** | Focus testing on the specific Android attack surfaces called out in each control |
| **Engineering Managers** | Use as acceptance criteria for security stories and compliance evidence |

## Scope

- Covers **Android 10 (API 29) through Android 17 (API 37)** with notes on version-specific behaviors
- References the **Android SDK, AndroidX/Jetpack, Google Play Services, and Google Tink** libraries
- Maps to **OWASP Mobile Top 10 2024**, **NIST SP 800-163**, and **CIS Android Benchmarks**
- Covers **native (Kotlin/Java)** apps; Flutter/React Native/Xamarin apps should also apply their framework-specific guidance

## Control Groups

| # | Group | Controls | Focus |
|---|---|---|---|
| 1 | [MASVS-STORAGE](Document/05-MASVS-STORAGE.md) | 2 controls, 18 sub-requirements | Secure data storage at rest |
| 2 | [MASVS-CRYPTO](Document/06-MASVS-CRYPTO.md) | 2 controls, 16 sub-requirements | Cryptography and key management |
| 3 | [MASVS-AUTH](Document/07-MASVS-AUTH.md) | 3 controls, 18 sub-requirements | Authentication and authorization |
| 4 | [MASVS-NETWORK](Document/08-MASVS-NETWORK.md) | 2 controls, 14 sub-requirements | Network communication security |
| 5 | [MASVS-PLATFORM](Document/09-MASVS-PLATFORM.md) | 3 controls, 22 sub-requirements | Platform interaction and IPC |
| 6 | [MASVS-CODE](Document/10-MASVS-CODE.md) | 4 controls, 18 sub-requirements | Code quality and supply chain |
| 7 | [MASVS-RESILIENCE](Document/11-MASVS-RESILIENCE.md) | 4 controls, 20 sub-requirements | Anti-tampering and anti-reverse engineering |
| 8 | [MASVS-PRIVACY](Document/12-MASVS-PRIVACY.md) | 4 controls, 18 sub-requirements | Privacy and data minimization |

**Total: 24 controls, 144 Android-specific sub-requirements**

## Training-Aligned Requirements

The [`requirements/`](requirements/) directory contains **93 additional testable requirements** derived from OWASP/Manicode mobile security training content, organized by MASVS control group:

| File | Requirements | Key Topics |
|------|-------------|------------|
| [MASVS-STORAGE-ANDROID.md](requirements/MASVS-STORAGE-ANDROID.md) | 13 | Keystore/StrongBox, EncryptedSharedPreferences, backup exclusion, FLAG_SECURE |
| [MASVS-CRYPTO-ANDROID.md](requirements/MASVS-CRYPTO-ANDROID.md) | 11 | AES-256-GCM, hardware key generation, key attestation, HPKE post-quantum |
| [MASVS-AUTH-ANDROID.md](requirements/MASVS-AUTH-ANDROID.md) | 12 | CryptoObject biometric binding, Credential Manager/passkeys, DPoP, OAuth |
| [MASVS-NETWORK-ANDROID.md](requirements/MASVS-NETWORK-ANDROID.md) | 8 | Certificate Transparency, TLS 1.3, Network Security Configuration |
| [MASVS-PLATFORM-ANDROID.md](requirements/MASVS-PLATFORM-ANDROID.md) | 14 | App Links, Intent security, WebView hardening, QR/NFC validation |
| [MASVS-CODE-ANDROID.md](requirements/MASVS-CODE-ANDROID.md) | 12 | Play App Signing, SBOM, SDK supply chain vetting, reproducible builds |
| [MASVS-RESILIENCE-ANDROID.md](requirements/MASVS-RESILIENCE-ANDROID.md) | 12 | Play Integrity API, R8 obfuscation, root/Frida detection |
| [MASVS-PRIVACY-ANDROID.md](requirements/MASVS-PRIVACY-ANDROID.md) | 11 | Scoped Storage, selected photos access, SDK consent gating, data deletion |

## Upstream Mapping

Every control in this document maps 1:1 to the upstream OWASP MASVS v2.x control of the same ID. Sub-requirements (e.g., `MASVS-STORAGE-1.3`) are Android-specific expansions that do not exist in the upstream standard.

## How to Contribute

1. Fork this repository
2. Create a feature branch
3. Submit a pull request with a clear description of the change
4. Reference the upstream MASVS control ID and any Android documentation links

## License

This work is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-sa/4.0/).

Based on the [OWASP MASVS](https://github.com/OWASP/owasp-masvs) by the OWASP Foundation.

## Acknowledgments

- [OWASP Mobile Application Security](https://mas.owasp.org/) project team
- [Android Security documentation](https://developer.android.com/privacy-and-security)
- [OWASP Mobile Top 10 2024](https://owasp.org/www-project-mobile-top-10/)
