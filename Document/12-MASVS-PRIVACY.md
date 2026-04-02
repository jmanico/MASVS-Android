# MASVS-PRIVACY: Privacy

## Overview

Android's privacy landscape has evolved dramatically — from the early days of broad permissions and unlimited identifier access to modern granular runtime permissions, scoped storage, restricted identifiers, privacy indicators, and the Privacy Sandbox. This category ensures Android apps practice data minimization, use privacy-preserving identifiers, provide transparent disclosures, and give users meaningful control over their data.

## Android Privacy Evolution

| Android Version | Key Privacy Feature |
|---|---|
| 6.0 (API 23) | Runtime permissions |
| 8.0 (API 26) | `ANDROID_ID` scoped per-app/user; background execution limits |
| 10 (API 29) | Background location separate; scoped storage; MAC randomization; no hardware ID access |
| 11 (API 30) | One-time permissions; auto-revoke unused permissions; package visibility restrictions |
| 12 (API 31) | Approximate location option; clipboard access notification; privacy indicators (mic/camera) |
| 13 (API 33) | Granular media permissions; per-app language; notification permission; clipboard sensitive flag |
| 14 (API 34) | Partial photo/video selection; selected photos access |
| 15 (API 35) | Screen recording detection; Private Space; partial screen sharing |
| 16 (API 36) | Local network permission; app-specific MediaStore versions |

## Google Play Data Safety Section

Required declarations:

| Declaration | Description |
|---|---|
| Data types collected | Location, contacts, financial, health, messages, photos, identifiers, etc. |
| Data shared | Which data types are shared with third parties |
| Security practices | Encryption in transit, data deletion mechanism |
| Third-party SDK data | All SDK data collection must be declared |
| Data deletion URL | Web-accessible deletion mechanism (required since Dec 2023) |

## Privacy Sandbox on Android

| API | Purpose | Status |
|---|---|---|
| Topics API | Interest-based advertising without cross-app tracking | GA |
| Attribution Reporting | Conversion measurement without cross-app identity | GA |
| Protected Audiences | On-device ad auctions | GA |
| SDK Runtime | Isolated SDK execution environment | Beta |

## Controls

| Control | Sub-Requirements | Focus |
|---|---|---|
| [MASVS-PRIVACY-1](../controls/MASVS-PRIVACY-1.md) | 7 sub-requirements | Data minimization and access restriction |
| [MASVS-PRIVACY-2](../controls/MASVS-PRIVACY-2.md) | 5 sub-requirements | User identification prevention |
| [MASVS-PRIVACY-3](../controls/MASVS-PRIVACY-3.md) | 5 sub-requirements | Transparency and disclosure |
| [MASVS-PRIVACY-4](../controls/MASVS-PRIVACY-4.md) | 6 sub-requirements | User data control and consent |

## OWASP Mobile Top 10 2024 Mapping

- **M6 — Inadequate Privacy Controls:** Directly addressed by all four controls
