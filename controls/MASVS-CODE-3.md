# MASVS-CODE-3

## Control

The app only uses software components without known vulnerabilities.

## Description

Android apps typically include dozens of third-party libraries via Gradle dependencies — networking (OkHttp, Retrofit), image loading (Glide, Coil), analytics, advertising, and more. Each dependency is potential attack surface. Known vulnerabilities in these dependencies can be exploited by attackers. This control ensures the app maintains awareness of its dependency tree and addresses known vulnerabilities.

## Android Sub-Requirements

### MASVS-CODE-3.1 — Scan Dependencies for Known Vulnerabilities

The build pipeline includes automated dependency vulnerability scanning using tools such as:
- OWASP Dependency-Check (`org.owasp:dependency-check-gradle`)
- Snyk
- GitHub Dependabot
- Google's `play-services-safetynet` / Play Integrity for runtime checks (limited)

Scans run on every build or at minimum on every pull request and release build.

**Rationale:** New CVEs are published regularly for popular Android libraries. Automated scanning catches known vulnerabilities before they ship to users.

### MASVS-CODE-3.2 — Maintain a Software Bill of Materials (SBOM)

The app maintains an SBOM listing all direct and transitive dependencies, their versions, and licenses. The SBOM is generated as part of the build process and stored alongside release artifacts.

**Rationale:** An SBOM enables rapid impact assessment when new vulnerabilities are disclosed. Regulatory requirements (EU Cyber Resilience Act, US Executive Order 14028) increasingly mandate SBOMs.

**Android References:**
- `./gradlew dependencies` — generates dependency tree
- CycloneDX Gradle plugin for SBOM generation in CycloneDX format

### MASVS-CODE-3.3 — Pin Dependency Versions

All dependencies in `build.gradle` use exact version numbers (e.g., `2.9.0`), not dynamic versions (`2.+`, `latest.release`). Gradle dependency locking (`dependencyLocking`) is enabled to produce reproducible builds.

**Rationale:** Dynamic versions can resolve to different (potentially vulnerable or malicious) versions between builds. Version pinning ensures reproducibility and prevents supply chain attacks via version hijacking.

**Android References:**
- Gradle dependency locking: `dependenciesLocking { lockAllConfigurations() }`
- `gradle.lockfile` — records resolved dependency versions

### MASVS-CODE-3.4 — Verify Dependency Integrity

Gradle dependency verification (`gradle/verification-metadata.xml`) is configured to verify checksums (SHA-256) and/or PGP signatures of all downloaded dependencies.

**Rationale:** Without integrity verification, a compromised Maven repository or MITM attack on the build could substitute malicious artifacts with matching version numbers.

**Android References:**
- `gradle/verification-metadata.xml` — configure with `./gradlew --write-verification-metadata sha256,pgp`

### MASVS-CODE-3.5 — Minimize Third-Party SDK Attack Surface

The app includes only essential third-party SDKs. Each SDK is evaluated for:
- What data it collects (for Google Play Data Safety Section compliance)
- What permissions it requires
- Whether it includes native code (`.so` libraries)
- Its maintenance status and security track record
- Whether it can be replaced with a platform API or smaller, audited alternative

**Rationale:** Every third-party SDK increases the attack surface, may collect data without the user's knowledge, and may introduce vulnerabilities. SDK supply chain attacks are an increasing concern (OWASP Mobile Top 10 2024 M2).
