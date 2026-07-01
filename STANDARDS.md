# BusHop Engineering Standards

This file documents which automation and design standards apply to this specific project.
It is the applied version of the universal standards in the project-retrospective-methodology repo.

**Universal reference**: `github.com/B67687/project-retrospective-methodology`

---

## Automation Standards Applied

### Tier 0 — Day 1 (present at project creation)

| Item | Status | How |
|------|--------|-----|
| CI build + test | ✅ | `.github/workflows/build.yml` — test + lint + assembleDebug on push/PR to main |
| Static analysis | ✅ | `detekt` configured + runs in CI |
| Dependency vulnerability scanning | ✅ | `.github/dependabot.yml` — weekly checks for Gradle + GitHub Actions |
| Secret scanning | ✅ | `gitleaks` runs as parallel CI job on push/PR |
| Signed commits | ✅ | All commits SSH-signed (`gpg.format=ssh`, key `~/.ssh/id_ed25519`) |
| Reproducible builds | ✅ | Dependency locking enabled; `settings-gradle.lockfile` generated |
| CHANGELOG | ✅ | Keep a Changelog format, retroactive for 41 releases |
| README skeleton | ✅ | Well-structured: features, stack, build, privacy, testing |

### Tier 1 — Within 10 Commits

| Item | Status | How |
|------|--------|-----|
| Conventional commits | ✅ | Enforced on PRs via `action-semantic-pull-request` |
| CHANGELOG presence CI check | ✅ | CI checks CHANGELOG.md is non-empty + warns if not modified in PR |
| Stats gate (derive from source) | ✅ | Badge auto-updated via CI after tests on main |
| Build provenance | ✅ | BuildConfig.GIT_SHA + BUILD_TIME + CI_RUN_ID embedded in APK |
| Code coverage gate | ✅ | JaCoCo 0.8.12 with 60% threshold configured; `jacocoTestCoverageVerification` in CI |
| Formatter enforcement | ✅ | `spotless` + `ktlint` — checked via CI |
| EditorConfig | ✅ | Present with LF, UTF-8, indent settings |
| SDK/toolchain pinning | ✅ | `gradle-wrapper.properties` pins Gradle 9.5.1; JDK 17 specified |
| Signed release tags | ✅ | `tag.gpgSign=true`; `git tag -s` uses SSH key |
| Concurrency-safe state design | ⚠️ Partial | ViewModel has AtomicInteger for API status; rest is MutableStateFlow (thread-safe via channel) |

### Tier 2 — Within First Release (cumulative)

| Item | Status | How |
|------|--------|-----|
| Architecture tests | ✅ | 8 automated rules (ArchitectureTest.kt) |
| Search index tests | ✅ | 45 tests — fuzzy matching, Levenshtein, performance |
| Domain purity | ✅ | Zero framework deps, all tests are mock-free |
|| Release script | ⚠️ Partial | `scripts/release.sh` manual; `.github/workflows/release.yml` automates APK build + GitHub Release on tag push |
| APK verification | ✅ | `CheckAndRenameDebugApk` Gradle task |
| ProGuard/R8 | ✅ | 86-line ProGuard rules file |
| Network security | ✅ | TLS pinning, cleartext blocked, HTTPS enforced |
|| ADRs (Architecture Decision Records) | ✅ | `docs/adr/` — 3 ADRs covering multi-module layout, domain purity, version catalog |

### Tier 3 — Quality of Life

| Item | Status | How |
|------|--------|-----|
| Design decision records | ❌ **Missing** | No ADRs |
| Commit date alias | ❌ **Missing** | Not set up |
| Release notes from CHANGELOG | ❌ **Missing** | Manual `gh release create` |
| PR template | ❌ **Missing** | No template |
| Pre-commit hooks | ❌ **Missing** | Not configured |
| Multi-architecture CI | ❌ **Missing** | No CI at all |

---

## Design Standards Applied

This project follows the design hierarchy from `DESIGN_STANDARDS_HIERARCHY.md`.

### Axioms in practice

| Axiom | How BusHop applies it |
|-------|----------------------|
| **A1 Modularity** | 3-module clean architecture (app/domain/data). 25 .kt files across modules. |
| **A2 Data Flow Direction** | Unidirectional: UI → ViewModel → UseCase → Repository → DataSource → API. No back-edges. |
| **A3 Fail-Fast** | Network security config blocks cleartext. FileProvider exported=false. CancellationException rethrown everywhere. |
| **A4 Explicit Over Implicit** | Manual DI in MainActivity composition root (no magic). Architecture tests enforce import rules. |
| **A5 Parse-Don't-Validate** | `NetworkResult<T>` parsed at API boundary. Domain models are data classes (no invalid states). |
| **A6 Layered Dependencies** | app → domain + data → domain. Enforced by build.gradle.kts dependency declarations. |

### Meso contracts in practice

| Contract | How BusHop applies it |
|----------|-----------------------|
| **M1 Interface Surface** | `BusRepository` (30+ methods), `BusArrivalDataSource`, `UpdateChecker` — all properly abstracted |
| **M2 State Management** | ViewModel + StateFlow for reactive state; DataStore for persistence |
| **M3 Resource Lifecycle** | OkHttp client singleton; coroutine scopes tied to ViewModel lifecycle |
| **M4 Error Domains** | `NetworkResult<T>` for API; `Result<T>` for storage; cancellation rethrown |
| **M5 Module Boundaries** | 3 separate Gradle modules, architecture tests verify |

---

## Current Gaps (highest priority to close)

| Gap | Effort | Impact | Why it matters |
|-----|--------|--------|---------------|
| **CI: build + test** | 1h | Prevents untested code | 112 tests run nowhere automatically |
| **Static analysis (detekt/ktlint)** | 30m | Catches style bugs | No automated code quality enforcement |
| **Dependabot** | 5m | Known-vulnerability deps | Transitive CVEs invisible |
| **gradle.lockfile** | 5m | Reproducible builds | Dependency drift across machines |
| **Test count badge auto-update** | 30m | Stale documentation | 3 different numbers exist |
| **Conventional commits** | Discipline | Parseable history | Only 26% of commits use prefixes |
| **Signed commits + tags** | 15m | Untrusted history | No commit verification possible |
| **JaCoCo coverage gate** | 30m | Coverage regressions | No visibility into untested code |
| **Release automation** | 1h | Manual release steps | `gh release create` done manually |

---

## Version

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-07-01 | Initial: automation tiers 0-3 + design axioms applied after retrospective |
|| 1.1 | 2026-07-01 | Wave 1: CI workflow, Dependabot, JaCoCo plugin, STANDARDS.md updated |
