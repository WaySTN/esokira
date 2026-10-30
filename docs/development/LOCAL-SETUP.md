# Esokira — Local Development Setup & Environment Verification

> **Canonical path:** `docs/development/LOCAL-SETUP.md`  
> **Version:** `0.1.0`  
> **Prepared:** 30 October 2026 (Asia/Jakarta)  
> **Document status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Repository audit status:** `UNVERIFIED` — source/build/configuration files of the actual Esokira application have **not** been inspected as part of preparing this guide.  
> **Application test status:** `NOT_RUN` — no Esokira Flutter, Go, PostgreSQL, migration, API, or CI test has been executed by this documentation task.  
> **Scope:** Developer onboarding and reproducible *local/disposable* environment for R0/S-01, supporting R1–R3 as features become available. R4 remains discovery-only.  
> **Audience:** Product Owner, developer, AI coding agent, Go/Flutter engineers, QA/security reviewer.  
> **Baseline stack selected by owner:** Flutter/Dart (`frontend/`), Go (`backend/`), PostgreSQL, monorepo. Everything else follows its own decision gate.

---

## 0. Read this before running anything

This is a **safe setup playbook**, not evidence that an application or dev environment already exists. The paths `frontend/`, `backend/`, `docs/` describe the intended monorepo organization; concrete source files, task runners, `.env.example`, Docker Compose, OpenAPI, SQL migrations, and service entrypoints **must be discovered in the real checkout**. The presence of an example command in this Markdown is *not* an assertion that it works on the user's machine or that the relevant artifact has been approved.

**Rules that always apply:**

1. **Inspect first, mutate second.** Locate the real repo root; record working tree state; inspect installed tooling and actual manifests. Do not scaffold over existing source.
2. **Owner decisions are not silently approved.** The choices Flutter, Go, PostgreSQL, monorepo are `DECIDED` in the planning context. Framework integrations, tool versions, local DB provisioner, Compose, IdP, migration tool, runtime variables, binding ports, and deployment environment are `PROPOSED`/`TBD` until approved or demonstrated in a real repository.
3. **Synthetic data only.** No production dumps, personal transactions, passwords, API keys, session tokens, or identifiable finance data in local fixtures, prompts, source, screenshots, CI logs, chat, or version control.
4. **No destructive automation.** Never run `DROP DATABASE`, reset/truncate, `docker compose down -v`, migration rollback, or delete data until an operator has verified a *disposable* target, read the impact, and explicitly authorized the action.
5. **No fake results.** A recommended test is `NOT_RUN`; an installed SDK is not a passed build; a running database is not proof of tenant isolation; a documentation review is not evidence that a release is ready.
6. **One bounded authorized task.** Respect the work order and Definition of Ready/Done. Do not run all modules or install all optional dependencies during initial setup.

**Stop and report `BLOCKED` for the affected step** if a required approval, checkout, credential policy, database target, tool version, or authoritative schema is unknown. Continue harmless discovery and documentation where possible.

## 1. Sources of truth and precedence

This file **orchestrates setup**. It does not own financial formulas, identity, database models, APIs, security controls, or implementation approvals.

| Need | Authority |
|---|---|
| Project entrypoint and documentation routes | [README](../../README.md) → [Documentation Index](../INDEX.md) |
| Global AI instructions | [Root AGENTS](../../AGENTS.md); for code also [Flutter AGENTS](../../frontend/AGENTS.md) or [Go AGENTS](../../backend/AGENTS.md) |
| Stack selection vs candidate tools | [ADR-0001 Tech Stack](../architecture/ADR/0001-TECH-STACK.md) and [Decision Log](../planning/DECISION-LOG.md) |
| Repository architecture boundaries | [System Architecture](../architecture/SYSTEM-ARCHITECTURE.md), [Backend Architecture](../architecture/BACKEND-ARCHITECTURE.md), [Frontend Architecture](../architecture/FRONTEND-ARCHITECTURE.md) |
| Database design, migrations, ownership | [Database Design](../architecture/DATABASE-DESIGN.md), **actually applied approved migrations** if any |
| HTTP contract | [API Guidelines](../architecture/API-GUIDELINES.md), **approved OpenAPI artifact** if any |
| Secrets, authentication, privacy | [Security & Privacy](../security/SECURITY-PRIVACY.md) |
| Money/domain behavior and golden numbers | [Financial Domain Rules](../financial/FINANCIAL-DOMAIN-RULES.md), [Calculation Specs](../financial/CALCULATION-SPECS.md), [Financial Test Cases](../financial/FINANCIAL-TEST-CASES.md) |
| Backlog, execution status, review | [Master Roadmap](../planning/MASTER-ROADMAP.md), [Tasks](../planning/TASKS.md), [Progress](../planning/PROGRESS.md) |
| Execution SOP, test matrix, exit gates | [Development Workflow](./DEVELOPMENT-WORKFLOW.md), [Testing Strategy](./TESTING-STRATEGY.md), [Definition of Done](./DEFINITION-OF-DONE.md) |

**Conflict rule:** an approved requirement governs intended behavior; a verified applied migration/contract describes the currently implemented shape; neither may silently override the other. Record any drift with affected file, ID, consequence, and decision required.

## 2. Scope, expected outputs, and initial readiness

### 2.1 Goal of a successful local setup

A successful *R0 local-setup task*, **when repository code actually exists and relevant decisions are approved**, should let another developer reproduce these results:

- Confirm project root, branch/working tree, and selected toolchain versions.
- Confirm Flutter and Go manifests/entrypoints (or document that they are not yet present) and the applicable SDK constraints.
- Provision/connect to **isolated development and test PostgreSQL** using authorized methods; verify DB name/user/host without displaying credentials.
- Apply **approved** migrations through the repository's actual migration runner; verify the effective schema and safe recovery method.
- Start Go API and Flutter application through **verified entrypoints**, with local-only permissions and no assumed endpoints.
- Perform relevant checks; preserve commands, actual exit status, sanitized output, environment and evidence.
- Demonstrate clean recovery/restart on the disposable environment **if in scope**, not on real user data.
- Update the actual task and [PROGRESS.md](../planning/PROGRESS.md) after a real repository audit and observed results.

If source code is absent, **audit + documented prerequisites** may be the honest completed output of a documentation-only task; it is not `DONE` for `TK-S01-01` through `TK-S01-14`.

### 2.2 Initial status snapshot

| Area | What is selected or known | Status requiring evidence/decision |
|---|---|---|
| Flutter mobile + Dart | Selected owner stack | Installed version, Android/iOS target, `pubspec.yaml`, lockfile, run configuration `UNVERIFIED` |
| Go backend | Selected owner stack | `go.mod`, version constraint, entrypoint, service port and test commands `UNVERIFIED` |
| PostgreSQL | Selected owner database | Server/client version, local provisioning, roles, DB names, schema migrations `UNVERIFIED` |
| Monorepo | Selected owner project organization | Actual checkout, file tree, Git state `UNVERIFIED` |
| Modular monolith, REST/OpenAPI | Architecture direction / proposal | No production route or contract assumed approved |
| Authentication, money encoding, ledger | Decisions `Q-02`, `Q-03`, `Q-04` | Approval not recorded in current Decision Log; protect affected implementation steps |
| Local/staging/CI data policy | `Q-10` and security requirements | Provisioning and retention policy require a recorded decision |
| Compose, container engine, migration CLI | Candidate implementation conveniences | Do not mandate or install without inspect/approval |
| Runtime tests | None executed by writing this Markdown | `NOT_RUN` |

## 3. Pick a local development mode explicitly

| Mode | Use when | Requirements | Never assume |
|---|---|---|---|
| **A — Native tools** | Workstation already has Flutter, Go and locally managed PostgreSQL | OS-specific installations, isolated DB role/data, approved configuration | Port `5432` available; database user is `postgres`; automatic migrations |
| **B — Host Flutter/Go + containerized PostgreSQL** | Owner permits Docker/other local container engine, **and actual Compose or equivalent config exists** | Approved image/version, data volume policy, mapped port, health check, secrets | That Docker is required or that `compose.yaml` exists |
| **C — Dev containers / integrated services** | Specific repository config is present and owner-approved | Files that are actually checked in, safe volumes, dependency and secret review | That `devcontainer.json`, Makefile, or scripts exist |
| **D — CI disposable environment** | A CI task is approved and pipeline config exists | Ephemeral DB, synthetic fixtures, pinned toolchains, no production credentials | CI vendor, container image, service name, or coverage threshold |

**Recommendation without auto-approval:** for first R0 development, prefer the least complex mode already reproducible on the real machine. A local native database is legitimate; Docker is not automatically superior. Record the chosen mode and the reason in a decision/task evidence record.

## 4. Prerequisites and official installation references

These links identify **authoritative installation instructions**, not a globally approved Esokira version. Resolve project version constraints **from checked-in manifests/lockfiles and recorded decisions**, not from whatever is latest on a website. Do not update an already working SDK without a task and compatibility review.

| Tool | Role | Official reference | Version/source of truth |
|---|---|---|---|
| Git | Checkout and traceability | <https://git-scm.com/downloads> | Actual repository/OS tooling |
| Flutter SDK + Dart | Mobile application | <https://docs.flutter.dev/install> | Existing SDK constraint/lockfile + owner decision; Dart is bundled with Flutter |
| Android tooling | Android emulator / physical device | <https://docs.flutter.dev/platform-integration/android/setup> | Target Android platform and local device capability |
| Go toolchain | Backend and pure financial engine | <https://go.dev/doc/install> | Actual `backend/go.mod` `go`/`toolchain` directives + review |
| PostgreSQL server/client | Local/disposable data storage | <https://www.postgresql.org/download/> | Approved compatibility and schema/test plan |
| `psql` reference | Read-only connection diagnostics | <https://www.postgresql.org/docs/current/app-psql.html> | Verified server/role/host/port |
| IDE/editor | Editing, debugging | VS Code / Android Studio, as appropriate | Personal preference unless repository mandates tooling |
| Container engine (optional) | Isolated DB or development services | Official vendor documentation for approved engine | Only after owner/team approval |

**Platform notes:**

- **Windows:** PowerShell commands are provided below. Android apps can be developed on Windows with a configured Android SDK/emulator or physical device. Building/running iOS requires a compatible macOS/Xcode environment; do not promise iOS verification from Windows.
- **macOS:** Flutter/Go/PostgreSQL supported; Android requires Android toolchain; iOS requires Xcode, signing and platform dependencies where applicable.
- **Linux:** Flutter/Go/PostgreSQL supported; Android requires configured toolchain and virtualization/physical device where applicable. A full iOS simulator build is not available on Linux.
- **WSL:** Linux Go/PostgreSQL inside WSL and Windows Android tooling may communicate across network boundaries; treat it as a *mixed environment*. Confirm how hostnames/ports map instead of assuming seamless localhost access.
- Physical/emulated device choice must be explicitly recorded; an installed SDK does not guarantee a usable device.

## 5. Phase A — Inspect the actual checkout (read-only)

Use **one** shell family. The snippets below are safe discovery examples; if the checkout is not a Git repository, `git` commands may fail and the failure should be recorded rather than hidden.

### 5.1 Windows PowerShell (run from actual project root)

```powershell
Get-Location
git rev-parse --show-toplevel
git status --short --branch
Get-ChildItem -Force | Select-Object Name, Mode

# Presence checks only; False is an observation, not necessarily an error.
Test-Path .\README.md
Test-Path .\AGENTS.md
Test-Path .\frontend\AGENTS.md
Test-Path .\frontend\pubspec.yaml
Test-Path .\backend\AGENTS.md
Test-Path .\backend\go.mod
Test-Path .\docs\INDEX.md
Test-Path .\contracts\openapi.yaml
```

### 5.2 macOS/Linux / Git Bash (run from actual project root)

```sh
pwd
git rev-parse --show-toplevel
git status --short --branch
ls -la

for file in \
  README.md AGENTS.md frontend/AGENTS.md frontend/pubspec.yaml \
  backend/AGENTS.md backend/go.mod docs/INDEX.md contracts/openapi.yaml; do
  if test -f "$file"; then printf 'PRESENT  %s\n' "$file";
  else printf 'MISSING  %s\n' "$file"; fi
done
```

**Audit follow-up:** inspect *filenames and relevant non-secret config fields*, not credential contents. Look for real test scripts, actual Flutter flavors, backend entrypoint(s), migration framework, `.gitignore`, CI descriptors and package lockfiles. Do not dump `.env`, `.git/config` with embedded credential URLs, key files, or secret stores to agent logs.

### 5.3 Discovery record (fill using observations)

| Field | Record only observed value |
|---|---|
| Repository root / commit / branch | `UNVERIFIED` until inspected |
| Uncommitted changes | `UNVERIFIED` until `git status` |
| Flutter manifest + lock | `PRESENT`/`MISSING` and path; **not** assumed |
| Go module + toolchain requirement | `PRESENT`/`MISSING` and requirement |
| OpenAPI location + approval | `PRESENT`/`MISSING`, approval evidence if any |
| DB migrations/runner | Actual paths/commands or `UNVERIFIED` |
| Local config template | Actual path or `MISSING` |
| Device / host OS | Observed OS/target, not guessed |
| Next safe task | Exact `TK-*` ID, prerequisites and blocker |

**If the source tree is absent:** stop before running `flutter create`, `go mod init`, `docker compose`, or migration tools. A separate R0 scaffolding task (for example `TK-S01-01`) must authorize new source files and version selection.

## 6. Phase B — Verify installed tooling without altering the repo

### 6.1 Windows PowerShell

```powershell
Get-Command git, flutter, dart, go, psql -ErrorAction SilentlyContinue |
  Select-Object Name, Source

git --version
flutter --version
dart --version
flutter doctor -v
go version
psql --version

# Optional only if PostgreSQL client tools are installed:
Get-Command pg_isready -ErrorAction SilentlyContinue
```

### 6.2 macOS/Linux / Git Bash

```sh
command -v git || true
command -v flutter || true
command -v dart || true
command -v go || true
command -v psql || true

git --version
flutter --version
dart --version
flutter doctor -v
go version
psql --version
```

If a command is missing, record `NOT_INSTALLED`; do not mask the failure as a passed prerequisite. `flutter doctor -v` may reveal installed SDK paths and toolchain diagnostics; **sanitize path/user identity and any sensitive environment details before sharing output**. Prefer copy/paste of only non-sensitive findings.

### 6.3 Dependency/version policy

1. Read actual `frontend/pubspec.yaml`, `frontend/pubspec.lock` (if present), `backend/go.mod`, `backend/go.sum` and CI pins **without changing them**.
2. Compare to installed SDKs and published supported platform toolchains. If incompatible, record the mismatch and available remediation; obtain scope approval before upgrading SDK or editing constraints.
3. **Do not run** broad `flutter upgrade`, `flutter pub upgrade`, `go get -u ./...`, or installation scripts solely to make a quick example pass.
4. If `go env` or dependency fetching may use remote networks, apply approved proxy/security policies; avoid sending private module names or credentials in logs.
5. Record the OS, CPU architecture, SDK version/channel, Go version, PostgreSQL client/server versions, device ID/type, and selected environment mode for reproducibility.

## 7. Phase C — Configure local secrets and application settings

Configuration has two classifications:

| Classification | Examples | Storage rule |
|---|---|---|
| **Public client config** | Local API base URL, safe feature flags, display locale | May be bundled with Flutter only if explicitly designated public |
| **Backend secrets** | DB password/connection URI, identity provider private keys, encryption material, signing keys | Never in Flutter bundle, `--dart-define`, source code, tracked `.env`, screenshots, CI artifacts or prompt |
| **Non-secret backend config** | Development environment name, bind host/port, log level, worker disabled flag | Derived from actual code contract; choose safe localhost defaults only after review |
| **Sensitive user data** | Real transactions/goals, access tokens, profile identifiers, exports | Not permitted in local test fixtures or logs |

### 7.1 Where should variables come from?

1. Inspect a *real* environment template or typed configuration definition, if one exists.
2. Record required keys, acceptable values, defaults and validation behavior. Do not invent `DATABASE_URL`, `AUTH_PROVIDER`, `API_BASE_URL`, `JWT_SECRET`, or `PORT` as canonical keys just because they are common names.
3. If there is **no** config contract, propose one under the authorized R0 task, classify each field, and request approval for fields affecting identity, network exposure, logging or data storage.
4. Keep local secret files outside version control. Ensure `.gitignore`/secret scanning covers the names actually used; avoid relying only on `.gitignore` after a secret has been tracked.
5. Use distinct local development/test credentials and database names. Do not reuse cloud or production credentials.
6. Do **not** copy arbitrary `example.env` values into production, log fully expanded connections, or print `env` / `Get-ChildItem Env:` for debugging.

### 7.2 Example-only configuration inventory (not a real Esokira `.env`)

| Logical concept | Where it belongs | How to resolve |
|---|---|---|
| Runtime mode (`development`/`test`) | Go server | Read actual typed config and review accepted values |
| PostgreSQL endpoint/DB identity | Go server and DB test tooling | Approved local DB provisioning; secrets in local secret store |
| Authentication verification config | Go server | `Q-02`, applicable `SEC-*` approval; no private signing key in client |
| API endpoint for device | Flutter **public** runtime config | Verified backend bind address and emulator/device routing |
| Logging/privacy policy | Go and Flutter | Security & Privacy; no sensitive inputs/results/token in logs |
| Migration schema/namespace | Migration runner | Actual approved migration and database policy |

**No starter secrets are included in this document.** For an approved local disposable database, generate unique local credentials using an approved mechanism; never paste them into prompts or documentation.

## 8. Phase D — PostgreSQL: isolated development and test databases

### 8.1 Mandatory isolation gate

- Confirm whether PostgreSQL runs natively or in an approved container/service. Determine **actual** host, port, administrative/provisioning method and supported version.
- Use **separate databases/roles** for development and disposable integration tests. A test runner should not hold rights to create/delete production databases or connect to internet-facing PostgreSQL.
- **Deny production credentials and dumps.** No real bank exports, real user data, or prod backup restore into developer machines.
- Dev/test data may be reset only after verified isolated target and authorization. Schema modifications require the approved migration path; SQL examples in `DATABASE-DESIGN.md` are **not runnable production migrations**.
- Secure bind/network policy: loopback or explicitly approved local container network; avoid public database binding or permissive `pg_hba.conf` shortcuts.

### 8.2 Non-destructive checks (optional; use approved connection parameters)

```sh
# Checks the client binary only:
psql --version

# If pg_isready is installed and the approved local port is known:
# pg_isready -h 127.0.0.1 -p <APPROVED_LOCAL_PORT>

# Template only: replace values after checking the selected dev DB.
# psql -X -h <LOCAL_HOST> -p <APPROVED_PORT> \
#   -U <APPROVED_DEV_USER> -d <APPROVED_DEV_DATABASE> \
#   -c "SELECT current_database(), current_user, current_setting('server_version');"
```

The SQL quoted in the template is an **illustration**, not a ready-to-paste cross-shell command. If executing a read-only diagnostic, use the actual tool's documented quoting syntax and never embed passwords in shell history. Authentication errors should be recorded without revealing the credentials.

**Recommended safe verification sequence after provision:** `server reachable` → `authenticated disposable role` → `current_database/current_user verified` → `approved migration status` → `synthetic fixture` → `tests with cleanup only in approved disposable DB`. Each step can be `BLOCKED` independently; a successful connection is not proof that DB authorization or migrations work.

### 8.3 Container/Compose variant (conditional)

Only if a real approved Compose file and an authorized Docker-compatible tool are present:

1. Inspect the actual `compose.yaml`/`docker-compose.yml` and referenced environment file for published ports, image pins, persistent volumes and mount locations **without displaying secrets**.
2. Confirm the project/service name, DB user/DB names, health check and volume lifecycle before running it.
3. Use **the exact service names and commands from the inspected file**; do not assume `docker compose up -d db` works or that the service is named `db`.
4. Do not run `docker compose down -v` automatically. Volume removal is a destructive operation, even on localhost.
5. Record how the Go process connects from its actual namespace: `localhost` inside one container is **not** necessarily a different container or the host.

## 9. Phase E — Go backend: module, build, test and startup

**Do not invent** `cmd/api`, `main.go`, port `8080`, `Makefile` targets, router or DI framework. They exist only if observed.

### 9.1 Prerequisites for actual code commands

- `backend/go.mod` exists, Go version is compatible and dependencies can be fetched under approved policy.
- Applicable task `TK-S01-01`/`TK-S01-03` has permitted the code/bootstrap scope.
- Backend config contract is known; DB/auth migrations required by the target entrypoint are ready or the app supports a safe non-data bootstrap mode.
- The process will bind only to the intended local interface unless an explicit host-network exception is approved.

### 9.2 Conditional Go validation — PowerShell

```powershell
if (Test-Path .\backend\go.mod) {
  Push-Location .\backend
  try {
    go version
    go list ./...
    go test ./...
  } finally {
    Pop-Location
  }
} else {
  Write-Output 'BLOCKED: backend/go.mod is absent; do not invent the module.'
}
```

### 9.3 Conditional Go validation — macOS/Linux/Git Bash

```sh
if test -f backend/go.mod; then
  (cd backend && go version && go list ./... && go test ./...)
else
  printf '%s\n' 'BLOCKED: backend/go.mod is absent; do not invent the module.'
fi
```

The commands above are **templates for a real implementation checkout**. They may download Go dependencies or run tests with side effects. Inspect tests, module policy and test DB configuration first; execute only in an authorized local/disposable environment. Avoid `go test ./...` if test packages can accidentally contact real services; first establish isolated test config.

### 9.4 Actual API startup

1. Locate the approved Go entrypoint and task runner (e.g. by inspecting files under `backend/`; no location guaranteed).
2. Check declared configuration and app lifecycle. Use local-only binding and non-production secret material.
3. Run **the command actually supported by the repository**, capture sanitized startup/error output and process exit code.
4. If a health endpoint exists in the approved implementation, test its real route and expected response. Do not assume `/health`, `/healthz`, `/ready`, or a port number.
5. Verify graceful shutdown and resource cleanup on the selected environment; do not kill/unbind unrelated processes.
6. If auth/DB decision gates are unresolved, a minimal **non-financial shell** may be allowed by an R0 task; a fake balance API or production-looking transaction route is not.

**Security check:** do not report 'API ready' if ownership validation, idempotency, sensitive logging or migrations required by the in-scope task remain untested.

## 10. Phase F — Flutter client, Android device and run configuration

**Do not invent** Flutter project structure, flavor, `--dart-define` keys, selected state management, generated files or backend URL. Flutter is an untrusted client; authoritative monetary writes/calculations belong to Go.

### 10.1 Discover a supported device

```sh
flutter doctor -v
flutter devices
```

- If Android tooling is missing, follow the official Android setup reference and install only components required for the approved target.
- A physical Android device requires appropriate developer settings/USB permissions and often a network reachable by the device; use a test device and safe synthetic data.
- For iOS, a supported macOS/Xcode setup is necessary. Record `NOT_AVAILABLE` on Windows/Linux instead of fabricating a pass.
- `flutter run` requires an *actual* Flutter manifest, recognized device, suitable host configuration and an entrypoint.

### 10.2 Conditional checks — PowerShell

```powershell
if (Test-Path .\frontend\pubspec.yaml) {
  Push-Location .\frontend
  try {
    flutter pub get
    flutter analyze
    flutter test
  } finally {
    Pop-Location
  }
} else {
  Write-Output 'BLOCKED: frontend/pubspec.yaml is absent.'
}
```

### 10.3 Conditional checks — macOS/Linux/Git Bash

```sh
if test -f frontend/pubspec.yaml; then
  (cd frontend && flutter pub get && flutter analyze && flutter test)
else
  printf '%s\n' 'BLOCKED: frontend/pubspec.yaml is absent.'
fi
```

Run these **only after a repository review and authorization**. `flutter pub get` may fetch packages and update `pubspec.lock`; record changes with `git diff`, and do not automatically commit lockfile modifications or upgrade dependencies. Widget tests may require fonts/SDK/device assets; if absent, report the actual failure.

### 10.4 Startup on a device (template, not an Esokira command)

```sh
# Only once frontend/pubspec.yaml and an approved runnable device exist:
# flutter devices
# cd frontend
# flutter run -d <VERIFIED_DEVICE_ID>   # example placeholder
```

Use **only public configuration** for a client base URL or feature flag; never pass PostgreSQL credentials, signing keys, identity-provider private secrets, real API tokens or financial data through Flutter build defines, asset files or debug logs.

## 11. Connecting Flutter → Go safely on local networks

Always connect **Flutter to Go API**, not Flutter directly to PostgreSQL. Verify the real backend listen host, port, device networking mode, Android network-security config and any API base URL **before** trying a request.

| Client execution environment | Common network consideration | Verification required |
|---|---|---|
| Flutter app running on developer workstation (supported desktop/web dev only if authorized) | Loopback often points to same workstation | Check actual bound host/port, route and TLS policy |
| Android Studio stock Android emulator | `10.0.2.2` commonly reaches the development host from the Android emulator | Confirm emulator type, Go bind address and firewall; **not universal** across emulators |
| Android physical device | `127.0.0.1` points to the phone, not the workstation | Use a controlled approved LAN/reverse proxy/adb route; never expose PostgreSQL |
| iOS Simulator on macOS | Host loopback behavior depends on configuration | Verify actual simulator routing and approved HTTP/TLS policy |
| Containerized Go backend | Container loopback is not generally the Windows/macOS/Linux host | Use actual network/service names and mapped ports from approved config |
| WSL + Windows emulator | Host/WSL networking may require explicit forwarding | Check OS/network mode and port exposure; no default workaround claims |

**Never fix reachability by:** exposing all backend interfaces publicly, disabling auth, accepting arbitrary origins, disabling TLS globally, setting insecure Android policy for release builds, embedding server secrets in Flutter, or logging personal finance payloads. For development-only local HTTP, apply a **scoped explicit policy** if required by the platform and approved by security review; release configurations must remain appropriately protected.

A successful HTTP response from an unauthenticated health route proves **reachability only**, not access control, API conformance or finance correctness.

## 12. Migrations, schema drift and disposable fixtures

### 12.1 Source of truth for DB state

The authoritative **deployed** database state is the result of *actually applied, approved, versioned migrations*. `DATABASE-DESIGN.md` includes candidate SQL and model discussions; those are **not** permission to create final production tables. An imagined `backend/db/migrations` path is not proof a runner or schema exists.

Before executing a migration:

- Locate the actual migration config, tooling, ordering, versions and checksum/procedure.
- Confirm the schema/ledger/money/owner boundaries needed for the task are approved (`Q-02`, `Q-03`, `Q-04` and applicable `DB-Q*`).
- Verify **target DB name, role, host and network namespace**. Use disposable local/CI DB. Record a rollback/recovery method and how to detect partial application.
- Inspect migration content for destructive operations, implicit backfills, broad privileges and sensitive defaults.
- Execute through the repository's actual runner and record applied version + checks. **Do not fabricate the command** in this guide.

### 12.2 Fixture and test data rules

- Construct two or more synthetic distinct principals for future ownership-negative tests; never infer user ID from arbitrary request body/header.
- For actual ledger tests, use data with explicit opening-balance and transfer semantics only after `Q-03/Q-04` is approved; transfer is not income/expense and two posting sides must commit atomically.
- For idempotency, test repeated client requests and timeout/replay semantics according to the approved protocol; do not equate retries with fresh writes.
- For calculations, reference `CS-*` and independent `FTC-*` oracles; do not encode undocumented rounding, growth rate, locale or stochastic defaults.
- Real user records, real session tokens, bank statement imports and production snapshots are prohibited for local tests.
- Clean up through an **approved, target-verified** mechanism; retain only sanitized evidence and non-sensitive test summaries.

## 13. Verification matrix: claims allowed vs claims forbidden

| Step | Evidence needed for `PASS` | Still **not** proven by this step |
|---|---|---|
| Checkout audit | `git`/file tree observations, commit/ref, relevant manifests | Functional application or owner approval |
| Toolchain | Version output + platform diagnostic | Build compatibility with code not yet present |
| PostgreSQL connection | Authenticated access to **verified disposable** DB, server/role identity | Migrations, owner authorization, atomicity |
| Migration | Actual runner, applied migration version and schema inspection | Financial correctness or production migration safety |
| Go build/test | Executed command, exit code and sanitized summary | End-to-end API authz without real DB/identity tests |
| Flutter analyze/test | Executed command, exit code and actual test report | Real-device UX or server-side authorization |
| Flutter run | Observed app on named device with safe local setup | Financial feature correctness or release readiness |
| Go API smoke | Verified endpoint status and controlled shutdown | Privacy, money correctness, idempotency or contract pass |
| API contract | Approved OpenAPI vs implemented response, auth/error cases | Numeric golden correctness unless tested separately |
| Financial domain tests | Independent `FTC-*` oracle, inputs/outputs, applicable approved conventions | Unapproved policy or probabilistic model validity |
| DB concurrency/security | Integration test with isolated PostgreSQL + 2 users, replay/rollback/denials | Other untested data paths or production isolation |
| Documentation/ZIP QA | File exists, link/path check, ZIP integrity and byte comparison | **Any** Flutter/Go/PostgreSQL test pass |

### 13.1 Suggested order for later real R0 tests

1. Inspect repo and relevant task/decision approvals (`TK-S00-*`).
2. Verify pinned toolchains and backend/frontend manifests (`TK-S01-01`).
3. Verify independent disposable PostgreSQL (`TK-S01-02`).
4. Verify app shell/Go bootstrap only after real entrypoints (`TK-S01-03`, `TK-S01-04`).
5. Verify approved migration runner (`TK-S01-05`), then identity/owner controls when authorized (`TK-S01-06`).
6. Verify API contract, pure Go financial test harness and CI per their actual `TK-S01-*` work packages.
7. Repeat applicable checks in a clean checkout/CI environment before R0 exit signoff; do not imply R0 is released merely by setup.

Use [Testing Strategy](./TESTING-STRATEGY.md) for `TS-001`–`TS-040`, [Security & Privacy](../security/SECURITY-PRIVACY.md) for `SEC-001`–`SEC-026` and `ST-001`–`ST-075`, and [Definition of Done](./DEFINITION-OF-DONE.md) for `DOD-G01`–`DOD-G10`. Run **only** what is relevant and authorized, documenting `NOT_RUN`, `BLOCKED`, or `FAIL` honestly.

## 14. Troubleshooting and safe escalation

| Symptom | First non-destructive diagnostic | Do **not** do |
|---|---|---|
| `flutter`/`go`/`psql` not found | Check PATH, shell reload, installed SDK directory, `Get-Command` / `command -v` | Install random script or upgrade whole stack without approval |
| Flutter doctor reports Android toolchain issues | Check documented Android SDK/licenses/emulator status and target platform requirements | Claim mobile device tested when only web/desktop works |
| Emulator cannot reach local API | Confirm emulator type, address routing, actual backend listen host/port, Windows firewall, WSL/container boundaries | Open PostgreSQL publicly, disable auth, bind sensitive service to all interfaces without review |
| Go module mismatch | Inspect `backend/go.mod`, `go version`, project version pins and CI toolchain | Edit `go.mod`, upgrade all dependencies or disable checksum validation by default |
| `flutter pub get` fails | Inspect network/proxy policy, `pubspec.yaml`, lockfile, package source and exact stderr (sanitize) | Delete lockfile without understanding reproducibility |
| PostgreSQL connection refused | Confirm server running, host, port, role and DB target; non-secret `pg_isready` | Broaden network access or assume database name/password |
| `password authentication failed` | Verify approved local role, connection method and secret source without echoing password | Add password to Git, terminal command history, screenshots or chat |
| Migration version/checksum mismatch | Verify **correct disposable DB**, migration history, runner and expected approved revision | Fake migration table, force baseline, `down` or drop DB without authorization |
| Test suddenly reaches live service | Immediately stop the test, revoke exposed tokens if necessary and investigate environment/target | Continue test or accept a PASS because data happened to be returned |
| `go test -race` unavailable | Check platform/CGO/compiler capabilities and CI support | Claim race tests passed because unit tests succeeded |
| Mixed Windows + WSL path/network failure | Determine which OS owns each SDK, process, emulator and DB; confirm explicit translation | Hardcode LAN IP or disable firewall without limited authorization |
| iOS build requested from Windows/Linux | Record platform limitation and Mac/Xcode requirement | Claim that running Android verifies an iOS build |

### 14.1 When to report `BLOCKED` instead of retrying

- No real repository checkout / manifest / approved entrypoint.
- Unknown or potentially production PostgreSQL connection target.
- An auth/money/ledger/DB decision required by the step is still unapproved.
- A required secret must be disclosed to a coding agent to proceed.
- Tests depend on external service permissions, real user data, or irreversible migration.
- No safe way to reproduce a failing test without risking production or personal data.

**Blocked report minimum:** attempted step, actual observation (redacted), intended task ID, missing approval/artifact, safe alternatives, next authorized action. Do not invent a workaround that weakens security.

## 15. Cleanup, restart and recovery (disposable environments only)

A reproducible environment needs **controlled teardown**, not unconditional deletion.

1. Identify exactly which application processes, database and service volumes the setup created. Never terminate unrelated workloads.
2. Stop the Go/Flutter dev processes using their actual runner/IDE lifecycle, with graceful shutdown where supported.
3. Keep local secrets in designated storage; remove only stale synthetic test data through the approved fixture mechanism after target verification.
4. If rollback is requested, inspect migration reversibility and confirm the exact **test** database. Some schema changes require forward-fix rather than automated down migration.
5. Record any cleanup failure or leftover volume/service as `PARTIAL` with recovery steps. A 'clean setup' assertion requires a subsequent repeatable successful run.
6. Never run blanket `rm -rf` against uncertain paths, wipe all Docker volumes, drop shared DBs, or delete production backups as part of local setup.

## 16. CI parity and reproducibility contract

Local and CI should be similar **in relevant behavior**, not necessarily identical operating systems:

- **Pin or explicitly constrain** approved Flutter, Go, PostgreSQL and package versions from verified repo/decision records. Document changes through review.
- Use clean checkout; verify dependency resolutions/lockfiles and database schema/migration version; never assume local cached packages are present in CI.
- Use a **disposable synthetic PostgreSQL** service for integration/concurrency tests; do not use production data or production secrets.
- Verify Go unit/financial tests independently of an HTTP server for the pure engine, plus relevant DB/API tests with real isolation.
- Run Flutter analyze/widget tests using actual app manifests; run emulator/device tests in an authorized suitable runner and report platform coverage honestly.
- Use least privilege CI identities; isolate jobs, mask/redact environment variables and logs, restrict artifact retention; adopt only an approved CI vendor/hosting policy.
- Check secret scanning, dependencies, migration safety and owner authorization; do not assign a made-up coverage percentage or p95 latency target.
- Include setup reproduction instructions in CI task only **after** actual pipeline files have been inspected/approved (`TK-S01-*` and `EN-006`).

## 17. AI coding agent procedure for a local setup session

**Required SOP (one session, one bounded task):**

1. **Select:** choose exact approved `TK-S00-*` or `TK-S01-*` scope from [Tasks](../planning/TASKS.md). If approval is missing, limit work to safe inventory and questions.
2. **Inspect:** read root and scoped `AGENTS.md`, actual working tree/config/manifests, selected architecture and task references; identify already-existing work.
3. **Classify:** state observed environment, `DECIDED` vs `PROPOSED/TBD`, and precise `BLOCKED` steps; do not ask for all 223 open decisions unnecessarily.
4. **Plan:** describe files likely touched, network/secret/DB hazards, dependent tasks and reversible testing plan. No speculative dependency churn.
5. **Execute:** make one authorized change; use actual entrypoints and commands, keeping local synthetic data and sensitive outputs private.
6. **Verify:** run applicable commands, record exit codes and sanitized outputs; distinguish failure, not-run, infrastructure-blocked and pass.
7. **Review:** check `git diff`, created files, secret scanning, actual migration/config compatibility, R0 DoD and applicable security controls.
8. **Handoff:** update [Progress](../planning/PROGRESS.md) with linkable evidence and next step; only change task status if justified by real repository observations.

### 17.1 Required handoff record (copy and fill, **do not pre-mark PASS**)

```yaml
session: null                     # Real timestamp/identifier when executed
repository_commit: null           # Observed SHA, or null
host_os_arch: null                # Actual OS and CPU architecture
mode: null                        # A / B / C / D after owner authorization
task_id: null                     # Exact TK-* ID
owner_approval_ref: null          # Real decision reference; null if absent
scope: null
files_changed: []
observed_manifests: []
flutter_sdk: null
go_version: null
postgres_server_version: null
db_target_class: UNVERIFIED       # Never print passwords/connection URIs
backend_start_command: null      # Only actual working command
frontend_start_command: null     # Only actual working command
migration_command: null          # Only actual verified runner
checks: []                        # Each: name, command, status, redacted evidence
issues: []
remaining_blockers: []
implementation_status: UNVERIFIED
application_tests: NOT_RUN
reviewer: null
next_safe_action: null
```

**Schema is a documentation template, not an application API.** `UNVERIFIED`/`NOT_RUN` are intentionally truthful placeholders. Any successful `PASS` record needs an actual executed command, output/exit status, environment and relevant acceptance criteria.

## 18. Definition of Ready for local setup tasks

- [ ] A real repository root and intended branch are known, or a scoped fresh scaffold is explicitly authorized.
- [ ] Exact task ID, target release slice and relevant decisions are recorded.
- [ ] Installed toolchain version and repository constraints have been compared.
- [ ] Chosen local environment mode and non-sensitive network layout documented.
- [ ] Dev/test PostgreSQL isolation confirmed by authorized provisioning; no production data path.
- [ ] Actual config keys and secrets handling policy discovered or explicitly gated.
- [ ] Any migration command, schema choice or IdP setup has a reviewed approval and safe rollback/recovery plan.
- [ ] Test commands and pass/fail evidence destinations identified; no auto-DONE expectation.

## 19. Definition of Done for an actual setup task

- [ ] Correct code scope implemented only within the authorized `TK-*` task and applicable AGENTS instructions.
- [ ] Checkout and toolchain observations recorded with versions and commit/ref.
- [ ] Backend/frontend manifests and supported commands are **verified from actual files**; commands executed on the intended host.
- [ ] Disposable DB identity/role is verified, security boundary validated for in-scope testing and required approved migrations applied/tested.
- [ ] All relevant Go/Flutter/database/API/financial/security checks show **real** outcomes; missing tests are `NOT_RUN`/`BLOCKED`, not implicit PASS.
- [ ] No personal finance data, tokens, passwords, signing keys or sensitive outputs included in files/logs.
- [ ] Restart, cleanup and recovery path is documented for local disposable services if part of task scope.
- [ ] Relevant [DoD universal gates](./DEFINITION-OF-DONE.md) passed with evidence, reviewer/approval where required, and [Progress](../planning/PROGRESS.md) updated.

**Scope distinction:** documentation QA may meet its *document* DoD without proving the app exists. A real R0 setup task remains `UNVERIFIED`/`PARTIAL`/`BLOCKED` until evidence actually demonstrates its own acceptance criteria.

## 20. Decision and blocker map for R0 setup

| Decision/task | What must be verified or approved | If unresolved |
|---|---|---|
| `TK-S00-01` / `TK-S01-01` | Repo audit, monorepo layout, toolchain constraints | Inspect or propose scaffold; do not claim the application exists |
| `TK-S00-06`, `Q-10` | Local/dev/test data policy, secret handling, ops baseline | Avoid cloud/live-data path; constrain to safe discovery |
| `TK-S00-07`, ADR `STACK-*` | Tools, SDK pins, router/client/migration candidates | Do not auto-select frameworks or run arbitrary installers |
| `TK-S01-02` | Disposable PostgreSQL environment and scoped roles | DB-dependent tests `BLOCKED` |
| `TK-S01-03` / `TK-S01-04` | Verified Go/Flutter entrypoints, API URL, device/SDK | Startup `NOT_RUN`/`BLOCKED`; docs/toolchain checks may proceed |
| `TK-S01-05`, `DB-Q*` | Migration tool, schema, rollback plan | No irreversible schema or placeholder migrations |
| `Q-02` / `TK-S01-06` | Auth provider/principal verification, owner model | No private financial writes or false ownership claims |
| `Q-03` / `Q-04` | Ledger/money/rounding decisions | No actual-money schema, posting or golden-test policy guesses |
| `EN-004` R0 / `TK-S01-*` | Pure Go finance test harness boundary | Do not conflate this with R1/S-05 completed compound calculator |
| `EN-005` | R3 background-worker enabler | Not needed for R0 local setup unless separately approved |

**Owner action requested when blocked:** decide only the scope-sensitive choices above, with recorded rationale and effective date in [Decision Log](../planning/DECISION-LOG.md). Approval is not implied by writing this guide or saying “lanjut dokumentasi”.

## 21. Maintenance and change-control policy

- Keep commands synchronized with *real repository files* whenever Go/Flutter entrypoints, migration tools, SDK pins, environment schema, or CI runner change.
- Every newly confirmed command should state **working directory, OS/shell, prerequisites, expected **non-sensitive** success signal, failure signal and recovery**.
- Distinguish `PROPOSED` examples (this guide) from `VERIFIED` command records in actual R0 implementation sessions. Never replace historical evidence with a later plan.
- If another document conflicts with runtime reality, open a discrepancy/decision rather than silently editing requirements. If an approved decision changes, update ADR/Decision Log and affected architecture/API/DB/Testing docs before or with code.
- Do not publish passwords, internal hostnames, real user exports or private service endpoints in screenshots, Markdown, CI artifacts or agent summaries.
- When a new tool is introduced, include its rationale, pinning strategy, supply-chain review, tests, cleanup strategy and approval. Dependencies are not adopted merely by being mentioned.

## 22. Setup readiness form (fillable, truthful baseline)

Use this table at the **start of an actual implementation session**; fill only verified values.

| Checkpoint | Initial value (document-only) | Evidence required to update |
|---|---|---|
| Repo root and Git branch | `UNVERIFIED` | Observed checkout and `git status` |
| Frontend Flutter manifest | `UNVERIFIED` | Actual file and constraint |
| Backend Go module | `UNVERIFIED` | Actual `go.mod` |
| Android/iOS device target | `UNVERIFIED` | Actual `flutter devices`/Xcode info |
| PostgreSQL dev/test isolation | `UNVERIFIED` | Observed DB/role identity, approved isolated target |
| App configuration keys | `UNVERIFIED` | Existing template/typed config, decision reference |
| Auth/ledger/money decisions | `NOT_RECORDED` in current Decision Log | Owner decision IDs/approvals |
| Migration tool and current schema | `UNVERIFIED` | Approved actual migration and schema inspection |
| Go start command | `UNVERIFIED` | Verified executable entrypoint and startup evidence |
| Flutter start command | `UNVERIFIED` | Verified device, config, run output |
| Go/Flutter/DB/API tests | `NOT_RUN` | Exact commands, exit codes, sanitized logs |
| R0 ready/release claim | **Not demonstrated** | Full R0 DoD, Security and owner gate evidence |

## 23. References

### Official platform documentation

- Flutter installation: <https://docs.flutter.dev/install>
- Flutter Android development setup: <https://docs.flutter.dev/platform-integration/android/setup>
- Go installation and toolchain: <https://go.dev/doc/install>
- PostgreSQL installation: <https://www.postgresql.org/download/>
- PostgreSQL `psql` reference: <https://www.postgresql.org/docs/current/app-psql.html>
- Git download: <https://git-scm.com/downloads>

These official sources provide **generic platform guidance**. Esokira's actual versions, commands and security policies are governed by approved project decisions and observed repository artifacts.

### Esokira local documentation

- Entry and routing: [README](../../README.md) · [INDEX](../INDEX.md)
- Stack decision control: [ADR-0001](../architecture/ADR/0001-TECH-STACK.md) · [Decision Log](../planning/DECISION-LOG.md)
- Backend/mobile/database: [Backend Architecture](../architecture/BACKEND-ARCHITECTURE.md) · [Frontend Architecture](../architecture/FRONTEND-ARCHITECTURE.md) · [Database Design](../architecture/DATABASE-DESIGN.md)
- Delivery controls: [Tasks](../planning/TASKS.md) · [Progress](../planning/PROGRESS.md) · [Development Workflow](./DEVELOPMENT-WORKFLOW.md) · [Testing Strategy](./TESTING-STRATEGY.md) · [Definition of Done](./DEFINITION-OF-DONE.md)
- Security: [Security & Privacy](../security/SECURITY-PRIVACY.md)

---

> **Final disclaimer:** This file is a reviewed **documentation artifact** and safe execution guide. It supplies no secret values, approved provider, fabricated migration, guaranteed working run command, passed application test, or production deployment permission. Use an actual checkout and owner-approved task to turn the `UNVERIFIED` entries into observations.  
> **Document status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL` · **Application tests:** `NOT_RUN`.
