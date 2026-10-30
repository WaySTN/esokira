# Esokira — Deployment, Release Operations & Recovery

> **Canonical path:** `docs/operations/DEPLOYMENT.md`  
> **Version:** `0.1.0`  
> **Prepared:** `2026-10-30` (Asia/Jakarta)  
> **Document status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Intended scope:** R0 engineering foundation; R1 real-finance MVP; R2 deterministic planning; R3 advanced simulations. R4 is discovery-only.  
> **Implementation / environment status:** `UNVERIFIED` — no user repository, CI runner, cloud account, deployed services, production database, store listing, or release pipeline has been audited as part of this documentation task.  
> **Application deployment and test status:** `NOT_RUN`; nothing in this file asserts an actual deployment, a successful backup restore, a compliant production environment, or a shipped app.  
> **Approval:** `NOT_RECORDED` for `Q-10`, other relevant release/security decisions, hosting, SLOs, RPO/RTO, retention, provider selection and public-release authorization.

---

## 0. Executive deployment contract

This document is the **operations playbook and release-acceptance specification**, not a deployment command script. The foundational owner choices are Flutter/Dart for mobile, Go for backend, PostgreSQL for the primary database, and a monorepo with `frontend/` and `backend/`. The product has sensitive personal financial data. A technically successful build is **not** permission to process real users' data or promote to production.

1. **Verify reality first.** Inspect the checkout, CI definitions, build manifest, approved OpenAPI, applied migrations, running endpoints, cloud resources and identity model. Any missing item is `UNVERIFIED`/`NOT_CONFIGURED`, never imagined.
2. **Protect actual finance.** A failed release must not double-post income, expense or transfers, lose ledger movements, leak private records, or treat simulated results as actual money.
3. **No automatic policy approval.** Provider choices, deployment region, data residency/legal analysis, provider access, backup window, retention and incident contacts remain `TBD` until owner/security/operations review is recorded in the Decision Log.
4. **Environment isolation is mandatory.** Local/CI/staging use synthetic or explicitly approved sanitized data; no production dumps, tokens or exported financial records in agent contexts, source control, test artifacts or third-party diagnostics.
5. **Never run destructive actions by default.** Production migration down/rollback, `DROP`, reset, `TRUNCATE`, deletion of cloud resources or backups, DNS cutover, credential rotation, app-store publish, and production deployment require separately scoped human authorization.
6. **Report evidence, not aspirations.** `PASS`, `DONE`, `DEPLOYED`, `RESTORED`, and `RELEASED` require actual execution logs, reviewed outputs, dates, environment identity and approvals; otherwise use `NOT_RUN`, `PARTIAL`, `BLOCKED`, or `UNVERIFIED`.
7. **Do not turn example commands or diagrams into a binding implementation contract.** Select concrete steps only after verifying the live repository, provider and decisions.

**Scope note:** this document does not design the financial algorithms, invent exact API endpoints, pick a CI vendor, prescribe a Kubernetes cluster, promise an app-store approval, or provide legal advice. It defines acceptance controls and examples marked `PROPOSED`.

---

## 1. Sources of truth and document routing

| Question | Authoritative source / prerequisite |
|---|---|
| Product scope, releases and acceptance | [PRD](../product/PRD.md), [Feature Matrix](../product/FEATURE-MATRIX.md), [Master Roadmap](../planning/MASTER-ROADMAP.md) |
| Architecture and topology concepts | [System Architecture](../architecture/SYSTEM-ARCHITECTURE.md), [Backend Architecture](../architecture/BACKEND-ARCHITECTURE.md), [Frontend Architecture](../architecture/FRONTEND-ARCHITECTURE.md) |
| Selected vs proposed technologies | [ADR-0001 Technology Stack](../architecture/ADR/0001-TECH-STACK.md), [Decision Log](../planning/DECISION-LOG.md) |
| Real schema, ownership and migrations | [Database Design](../architecture/DATABASE-DESIGN.md), *verified applied migration files when present* |
| API behavior and compatibility | [API Guidelines](../architecture/API-GUIDELINES.md), *approved machine-readable API contract when present* |
| Security, privacy, retention and incident response | [Security & Privacy](../security/SECURITY-PRIVACY.md) |
| Money/financial integrity | [Financial Domain Rules](../financial/FINANCIAL-DOMAIN-RULES.md), [Calculation Specs](../financial/CALCULATION-SPECS.md), [Financial Test Cases](../financial/FINANCIAL-TEST-CASES.md) |
| Scenario and advanced release risks | [Scenario Engine](../financial/SCENARIO-ENGINE.md), [Monte Carlo](../financial/MONTE-CARLO.md), [Goal Optimizer](../financial/GOAL-OPTIMIZER.md) |
| Test gates and acceptance | [Testing Strategy](../development/TESTING-STRATEGY.md), [Definition of Done](../development/DEFINITION-OF-DONE.md) |
| Local workstation setup | [Local Setup](../development/LOCAL-SETUP.md) |
| Agent work and logs | [Root AGENTS](../../AGENTS.md), [Frontend AGENTS](../../frontend/AGENTS.md), [Backend AGENTS](../../backend/AGENTS.md), [Development Workflow](../development/DEVELOPMENT-WORKFLOW.md), [Tasks](../planning/TASKS.md), [Progress](../planning/PROGRESS.md) |
| Overall document map | [Documentation Index](../INDEX.md) |

Approved, verified machine-readable API contracts, deployment manifests and applied database migrations describe the **deployed state** if they exist; product and financial requirements describe the **intended approved behavior**. If they disagree, record `DOCUMENTATION_DRIFT` with evidence instead of treating either mismatch as silent approval.

## 2. Release model and environment boundaries

| Environment | Purpose | Data classification | External exposure | Typical approval |
|---|---|---|---|---|
| Local | Developer/agent inspection, debugging, unit tests | Synthetic, disposable | Loopback or safely scoped dev | Task owner; no production mutation |
| CI | Repeatable lint, build, tests, contract, security checks | Synthetic/ephemeral | Narrow runner permissions | Repository-maintainer approval for pipeline and secret scope |
| Staging | Integration, migrations, reliability rehearsal, mobile-client compatibility | Synthetic or sanitized **only under approved policy** | Restricted private access | Technical + security review of environment boundaries |
| Production | Real users and financial information | Personal/financial data | Public API/mobile clients where expressly authorized | Product Owner **and** applicable security, privacy, operations and legal sign-offs |

**Do not use staging as a secondary production copy.** Never share production DB passwords, JWT issuer credentials, API keys, logging sinks, object storage buckets, Apple/Android signing credentials, or sensitive financial fixtures across environments. Protect build/release roles separately from ordinary CI test roles.

### 2.1 R0/R1/R2/R3 delivery expectations

- **R0 (`S-01`)**: local reproducible environment, CI scaffolding, auth boundaries and financial test harness. A functioning isolated integration environment is useful; production infrastructure is not implied or required to claim *document readiness*.
- **R1 (`S-02`–`S-06`)**: real accounts, balances and transactions. Before real-user launch, authz, tenant isolation, ledger atomicity/idempotency, export/delete, backup/restore, telemetry privacy and incident workflows must be reviewed and tested.
- **R2 (`S-07`–`S-09`)**: simulation and planning. Scenario versioning/staleness, isolation from actual finance and reproducible results are release-blocking.
- **R3 (`S-10`–`S-12`)**: optimizers, workers, Monte Carlo and Financial GPS. Resource quotas, job durability/cancellation, approved stochastic/optimization assumptions and truthful risk disclosures become additional gates.
- **R4 (`FR-023/FR-024`)**: discovery only, not a deployment target without separately approved scope.

No dates, throughput, cloud price, app-store availability, RPO/RTO values or supported device matrix are approved by this document.

## 3. Decision and approval hierarchy

**Parent decision `Q-10` remains `NOT_RECORDED`.** Deployment-related unknowns include provider, region, network topology, authentication integration, backup/PITR, restore capabilities, data lifecycle, audit/telemetry retention, cloud access, CI provider, secret management, offline behavior, privacy review and incident owner.

Other critical upstream decisions:

- `Q-02` — identity provider, token lifecycle, service verification; must be resolved before handling private real-user traffic.
- `Q-03` / `Q-04` — ledger, exact-money representation and transaction semantics; must be resolved before production actual-finance writes/migrations.
- `Q-05`, `Q-06`, `Q-11` — assumptions, scenario versioning, goal funding; gate affected R1/R2 features, not unrelated infrastructure research.
- `Q-07`, `Q-08` — optimizer objective and stochastic model; gate R3 computation/public claims.
- `Q-09` — notification provider/consent; gate outbound messaging.
- Local `OPS-Q01`–`OPS-Q18` at §27 track deployment-level choices; they are **not** 18 extra approvals from the owner.

An agent may draft candidate manifests, tests and documentation in an explicitly isolated task. **It must stop** before enabling live personal-data processing, producing irreversible schema changes, changing network/public access or proclaiming production readiness without applicable approvals.

## 4. Inventory and topology discovery

Before proposing a platform, inventory **actual** assets (write `NOT_FOUND` when missing):

| Layer | What to discover | Expected output |
|---|---|---|
| Monorepo | real branches, build manifests, `frontend/`, `backend/`, lockfiles | Repo commit, locations, dependency versions |
| Go API | actual entrypoint, configuration loader, health handling, middleware, binary/process model | Verified command or `NOT_CONFIGURED` |
| Flutter | actual flavors, Android/iOS build targets, signing, app identifiers, client endpoint strategy | Target map or `TBD` |
| PostgreSQL | provider/version, DB roles, approved/applied migrations, extensions, PITR capability | Evidence of effective schema/roles; **never dump secrets** |
| Auth | IdP endpoints, issuer, token verification key rotation, app-user mapping | Approved provider-neutral contract or `BLOCKED` |
| Jobs | approved worker process and durable queue, retry, quotas, retention | R3 runbook if applicable, otherwise `NOT_IN_SCOPE` |
| CI/CD | pipeline files, build runners, secret scopes, protection rules, artifact store | Actual stages and privileged actions |
| DNS/TLS | domain, ingress, certificates, internal/private networks, exposure | Approved architecture and verification evidence |
| Observability | metrics, structured logging, tracing, alerts, redaction rules | Data inventory and links to dashboards if available |
| Recovery | backups, restore credentials, drill environment, ownership | Last successful **tested** recovery result or `NOT_RUN` |

**Topology candidates** (none selected): a single Go API process plus managed PostgreSQL for early phases; API and separately scaled worker from the same codebase for R3; environment-specific managed services for storage and telemetry. The backend being a *logical modular monolith* does **not** automatically prescribe one host/container, Kubernetes, AWS, GCP, Azure, Fly.io, Render, Docker Compose or a particular provider.

### 4.1 Diagram (logical, not a provisioned environment)

```text
Flutter mobile (Android / iOS where approved)
    │ authenticated TLS; public client is untrusted
    ▼
[approved TLS ingress / gateway / rate-limit boundary]
    │ authenticated principal + request trace
    ▼
[Go API: domain use cases / authorization / financial engine]
    │ protected DB identity and private transport
    ▼
[PostgreSQL: durable actual financial source of truth]
    │ backup / WAL / PITR according to approved policy
    ▼
[isolated encrypted recovery storage / restore process]

Optional R3 path: Go API → durable job metadata/queue → Go worker
                                       ↕
                          versioned scenario / optimizer jobs

Observability: allowlisted metrics + redacted logs + restricted alerts
```

**Not included by implication:** a public database endpoint, direct Flutter SQL access, externally callable worker internals, public object URLs for personal exports, or an analytics SDK with unrestricted user events.

## 5. Release artifact and provenance contract

For each releasable component, record at minimum:

| Field | Required record | Why |
|---|---|---|
| Release ID | Immutable identifier plus release train R0/R1/R2/R3 | Correlation across client/backend/schema |
| Git source | Repository revision/commit and reviewed branch/tag | Traceability |
| Artifact | Immutable digest/hash and storage location | Prevent unexpected rebuild/drift |
| Toolchain | Actual Flutter/Dart, Go, PostgreSQL/schema versions and lockfile checksum | Reproducibility |
| Supply chain | Dependency and license scanning results; unresolved issues | Risk review |
| Config baseline | Environment name, configuration revision, **secret references only** | Detect configuration drift without leaking secrets |
| API/DB compatibility | Approved contract revision and applied migration version | Safe old-mobile-client support |
| Review | Build/test evidence, change approval and release approver | No anonymous promotion |
| Rollout | Deployment time, environment, batch/canary state (if selected), health | Operational audit |
| Recovery | Previous compatible artifact, restore/forward-fix plan, contact | Failure response |

Suggested version fields should be **designed and approved** before use; do not hardcode release IDs or fake hashes in production logs. Store immutable artifacts with least-privilege access and retention agreed for financial/privacy data classification.

## 6. CI pipeline architecture (PROPOSED; do not assume configured)

A representative stage model. The **tool and runner** must be chosen after inspecting the actual repository:

1. **Source integrity:** review authorized change, detect tracked secrets, check protected branch/review policy.
2. **Format/lint/static:** Go formatting/static checks; Dart analyzer/formatter; dependency and license policy checks.
3. **Pure tests:** Go financial unit/golden/property tests, Flutter unit/widget tests, schema/API validation where artifacts exist.
4. **DB integration:** provision disposable PostgreSQL with app-equivalent role, apply approved migrations, test constraints/tenant isolation/concurrency/idempotency, then tear down disposable resources.
5. **API/consumer:** approved OpenAPI validation, backend contract tests, Flutter client compatibility/error mapping, version-skew checks.
6. **Security/privacy:** appropriate authn/authz negative tests, SAST/dependency/secret findings triage, logging PII scrub, R1 export/delete and R3 worker quotas when in scope.
7. **Build:** reproducible Go artifact and Flutter platform artifact with provenance; signing via protected secrets only on authorized release jobs.
8. **Staging rehearsal:** deploy in restricted environment, smoke flows, prove migration + backward compatibility, run failure/recovery drills appropriate to risk.
9. **Manual gate:** record Product Owner/technical/security/privacy approval required for target environment; approvals cannot be simulated by green CI.
10. **Controlled rollout:** deploy immutable reviewed artifacts, monitor, compare release metrics, pause/revert code where safe; document post-release evidence.

**Failure policy:** a failed mandatory gate blocks promotion; `NOT_RUN` ≠ `PASS`; a waiver requires explicit approver, scoped risk and expiry where permitted. Financial ledger corruption, cross-user data access, lost secrets or untested restore cannot be waived into a safe production release.

### 6.1 Branch/PR threat surface

- Untrusted pull requests must not receive production signing keys, DB credentials, cloud privileged tokens, or production secret stores.
- Dependabot/third-party code updates need review; scripts run with narrow permissions and pinned source provenance where feasible.
- CI test artifacts cannot contain raw customer data, tokens, `.env`, SQL dump contents, or unredacted financial traces.
- Release jobs should use least-privilege short-lived credentials and explicit environment protection where supported; approach is TBD by provider.

## 7. Configuration, secrets and environment matrix

| Config category | Example logical property (not an environment variable name) | Classification | Gate |
|---|---|---|---|
| API endpoint | Backend origin and expected TLS server name | Public client config but integrity-sensitive | Build per environment, no hardcoded production values in tests |
| Identity validation | Issuer/audience/JWKS rotation and mapping rules | Security-sensitive; public keys ≠ private keys | Identity decision and token-validation tests |
| Database access | Host, role, TLS, pool sizing, password | Private/secret | Secret manager/env injection with least privilege |
| Auth/session | Signing key, trusted redirect URIs, session settings | Secret or security-sensitive | Approved policy, rotation, no shared credentials |
| Worker | Queue endpoint, job lease, concurrency/timeouts | Private/secret | R3 approved quotas, durable retries, tenant isolation |
| Telemetry | Log destination, sampling, redaction, exporter keys | Private and privacy-sensitive | Data inventory and privacy review |
| Storage/backup | Bucket/project, encryption key ref, lifecycle and restore identities | Private/secret | Backup/restore controls, retention and drill |
| Mobile signing | Android/iOS signing identity and provisioning key | Restricted secret | Store release owner + protected signing process |
| Feature flags | Enabled modules/releases and compatibility | Security/behavior-sensitive | Safe defaults; R3 cannot become production by flag alone |

**No actual variable names, values, URLs, credentials, storage paths or cloud IDs are invented in this document.** Use the real approved config schema if available, otherwise file a task to design one. Keep a redacted inventory of which environments use each secret without storing the secret itself. Never send secrets to the AI agent's chat or commit history. Rotating a leaked credential includes revocation, audit of use, replacement and incident assessment; merely deleting the line from Git is insufficient.

## 8. Network, ingress, authentication and authorization gates

- Public Flutter clients connect to an authenticated TLS endpoint. No plaintext production HTTP or `skip certificate verification` bypass to unblock testing.
- PostgreSQL and worker internals are not publicly exposed; restrict ingress/egress to documented needed peers and approved management paths.
- Request identity is verified in Go. A valid session token alone does **not** authorize another user's account, child resource, scenario result or export.
- Apply owner-scoped authorization on nested resources, job metadata, result retrieval, mutation, export, deletion and diagnostic support actions.
- Rate limiting, payload size limits, request timeouts and budgets must be sized after measured workloads/security review. Do not invent values and claim they meet SLOs.
- Logs and metrics must not leak financial numbers, exported records, access tokens, headers, raw request bodies, JWT contents or seed-derived private sample paths.
- Define a trusted proxy/forwarded-header policy appropriate to the selected network. Never trust arbitrary client-supplied `X-Forwarded-*` headers as evidence of internal identity.
- Align token/session expiry behavior with old mobile versions and clock skew; fail closed on missing/untrusted identity.
- Review TLS certificates, rotation/expiry and DNS or ingress changes, record the actual verification command and result.

## 9. Health checks and startup/shutdown behavior

| Probe / condition | Required semantics | Notes |
|---|---|---|
| Process liveness | Process responds or watchdog sees it running | Must not reveal DB credentials, user info or configuration |
| API readiness | Ready only when dependencies required to safely serve the route are usable | Don't return ready if incompatible schema or required identity verification is unavailable |
| Migration compatibility | Running code's minimum/maximum compatible schema version is accepted | Do not auto-run destructive migrations on every API process start |
| Background jobs | Worker readiness includes queue/lease and resources if worker is deployed | Distinguish worker outage from API health |
| Shutdown | Stop new work, finish/rollback in-flight work safely, release DB connections and leases | Avoid duplicate ledger posting and orphaned jobs |
| Synthetic smoke | End-to-end test with isolated synthetic tenant, no real-user posting | Protect against durable test pollution |

**Endpoints and paths are not prescribed here.** Actual health checks must be discovered in approved API/run configurations. Health probes must be network-restricted and not enumerate data or users.

## 10. PostgreSQL provisioning and privilege boundary

Before any production DB resource exists, approve: provider/version/extensions, network access, user roles, application/migrator/read-only privileges, isolation and auditing, encryption transport/rest, backup/PITR, maintenance, retention, credentials and rotation.

- Separate **application role** (least privilege) from **migration role**; do not run API as a superuser or a role that bypasses tenant protections without explicit reviewed design.
- Provision a disposable integration DB with the **same enforcement style/role** needed to test ownership and RLS (if selected). Tests as superuser do not establish production isolation.
- Database schema is defined by **applied and reviewed versioned migrations**, not example DDL in Markdown.
- Record extension/version compatibility, max connections, migration/checkpoint performance and operational limits based on measurement.
- Maintain access-control review for admins, support and recovery staff; backup privilege can expose entire user data sets.

## 11. Database migration and release compatibility

**Prefer a staged expand → migrate/validate → contract sequence when compatibility is needed.** This is a strategy, **not** an assertion that a particular migration runner has been chosen.

1. **Preflight:** identify exact release, migration plan and DB version; inspect backup/restore availability; evaluate data volume, blocking locks, query plans, live app client versions and approval. Check for irreversible DDL/data rewrites.
2. **Expand:** add compatible columns/tables/indexes/constraints cautiously. Preserve old API behavior where old mobile clients may still connect. Measure/mitigate locking and disk risks.
3. **Backfill:** if required, use explicitly approved resumable batches with checkpoints/reconciliation. Preserve precise money representation and idempotent updates. Don't use unrestricted full-table rewrites without review.
4. **Validate:** compare counts, reconciliation totals, constraints, ownership boundaries, actual money invariants and contract tests; track migration revision on each environment.
5. **Cutover:** deploy compatible Go API/client changes with approved ordering; monitor failures/throughput and run synthetic smoke.
6. **Contract later:** remove legacy fields only after proving old clients/rollback paths no longer rely on them; obtain separate destructive-change authorization.

**Mandatory financial migration checks:** existing opening balance semantics; transfer sum neutrality; reconciliation of account/net worth state; no duplicate idempotency records; cross-user isolation; financial results still satisfy approved `DR-*`, `CS-*`, `FTC-*`. If a model choice `Q-03/Q-04` is unresolved, finalizing the persisted schema is `BLOCKED`, not a reason to guess.

**Never promise that schema rollback is safe.** Where data has been transformed or old clients are still live, the correct incident route may be forward fix or application rollback *with compatible schema*, not a blind migration-down command.

## 12. Backup, PITR, RPO/RTO and restore drills

| Concern | Required evidence before real-user release | Approval status now |
|---|---|---|
| Recovery objectives | Measurable owner-approved RPO and RTO for scoped data/services | `TBD` |
| Backup schedule | Frequency, backup class (logical/physical), coverage, automatic failure alerts | `TBD` |
| Point-in-time recovery | Whether WAL/PITR is supported and verified, gap/retention boundaries | `TBD` |
| Encryption | At-rest and transit controls, key ownership/rotation, restricted storage | `TBD` |
| Retention | Data/backup/log retention, deletion lifecycle, legal holds after specialist review | `TBD` |
| Restore permissions | Named/reviewed operators with logged access; no public dump URLs | `TBD` |
| Restore drill | Timestamped isolated restore, sample reconciliation, integrity/tenant checks, measured elapsed recovery | `NOT_RUN` |
| Cross-region/offsite | Risk-based review and residency/legal implications, not an automatic requirement | `TBD` |
| Failure handling | Alert when backup or restore is impossible, owner and escalation | `TBD` |

### 12.1 Restore rehearsal — safe procedure

1. Identify **nonproduction target** clearly; require explicit authorization. Do **not** overwrite an existing production DB.
2. Select a backup with known timestamp and coverage; verify file/source integrity and secret access without logging raw data.
3. Restore to an isolated environment with restricted principals and network. Synthetic input is preferred; handling any real backup requires a separately approved policy and personnel.
4. Confirm DB version, migrations, constraints, row counts, user-scoped authorization, critical aggregates and sampled exact-money reconciliations.
5. If recovering from a point in time, document the cutoff and un-restored writes; do not claim zero loss without evidence.
6. Measure actual elapsed recovery; compare to **approved** RTO/RPO, not self-invented defaults.
7. Check replay/idempotency, export/deletion tombstones, consent/retention effects and job queues to prevent resurrected or duplicate behavior.
8. Capture a **redacted** drill report: backup ID/hash, versions, timestamps, operator, checks run, PASS/FAIL and remediation.
9. Dispose of restored data in compliance with the approved retention/deletion policy; revoke temporary access.

**Hard stop:** no production-readiness claim without a successful restore drill and policy approval appropriate to the planned launch. A backup job returning success is **not** proof that recovery works.

## 13. Release deployment sequence and rollout options

1. **Pre-deploy sign-off:** exact scope, release train, test artifacts, compatibility matrix, rollback/forward-fix plan, data changes and owner sign-off verified.
2. **Freeze and identify artifacts:** reviewed Git revision, immutable API/image/build digest, migration revision, Flutter version/build numbers, secret/config revision.
3. **Stage rehearsal:** deploy that *same* artifact to staging where compatible, run migration rehearsal, old-client contract tests, security and smoke suite.
4. **Gate production:** obtain explicit release authorization, not merely a comment that documents are complete. Escalate any P0/financial/privacy blockers.
5. **Protect DB:** capture recent backup/recovery readiness; validate compatible migration plan; ensure one controller executes schema changes as designed.
6. **Roll out API:** controlled release using approved provider capabilities; optional rolling/blue-green/canary only if architecture actually supports it.
7. **Health/readiness:** verify API, auth, DB access, tenant isolation smoke, idempotent synthetic flows, error redaction and key user journeys.
8. **Observe:** compare error, latency, auth failures, transfer issues, migration locks, worker failures and cost against approved baseline.
9. **Decision:** continue, pause, rollback code when safe, or forward-fix with operator/incident approval; **never auto-reverse committed financial writes**.
10. **Handoff:** record version, change evidence, residual risk, approvals, actual release status and next monitoring owner in `PROGRESS.md`.

Rollout technique is `TBD`: do not claim **zero downtime** or use a blue-green term unless schema compatibility, routing, data synchronization and reversal are tested in the real provider.

## 14. Rollback, forward-fix and emergency stop

**Four different kinds of reversal must not be confused:**

| What failed | Potential response | Guardrail |
|---|---|---|
| Stateless API code | Revert to previous compatible immutable artifact | Confirm DB schema and mobile-client compatibility first |
| Flutter client release | Pause rollout or publish corrected version via platform process | Installed clients may not downgrade; server must support version skew |
| DB migration or backfill | Forward fix / compatible migration; restore only through authorized disaster workflow | Migration-down can destroy or corrupt actual data; do not run blindly |
| Actual financial record | Domain-approved correction/reversal use case with audit | **Never** treat ledger posting as a deploy transaction to be blindly rolled back |
| Background jobs | Pause intake, cancel leases safely, dedupe resumed work | Avoid partial/untraceable simulation results or duplicate side effects |
| Security exposure | Restrict ingress, revoke/rotate affected identity/keys, incident flow | Preserve evidence safely and perform privacy/legal review |

**Emergency stop policy:** maintain a reviewed process to disable risky features and writes without falsely reporting successful operations. Fail closed for unauthenticated/unauthorized access, preserve auditable state, and never convert a client timeout into an automatic duplicate write.

## 15. Flutter mobile release and backward compatibility

This section covers **both Android and iOS only if approved as target platforms**; the final store strategy and minimum SDK versions are `TBD`.

- Verify the actual Flutter project, bundle/application identifiers, release flavors, permitted permissions, icon/branding approvals, signing/provisioning and build numbers in the checkout.
- Keep signing keys/certificates and store credentials in protected release systems; never put them in source, chat, public CI logs or test fixtures.
- Separate `debug`, `staging` and `production` API origins using approved build configuration; do not ship debug bypass, insecure transport or internal testing URLs.
- Confirm mobile client/auth flow compatibility when API changes; old installed versions may remain in circulation after server deploy.
- Test install/upgrade, session refresh, offline/timeout recovery, network switching, screen privacy, accessibility and key financial UI states on approved target devices/emulators.
- Run app privacy/data-collection disclosure and applicable store policy checks with responsible reviewers; third-party SDK telemetry may require consent/disclosure.
- An app-store upload **is not** equivalent to public availability; publication is asynchronous and depends on provider review and regional availability.
- For app/store rollback, use an approved halt/roll-forward process; do not assume a released mobile app can be forcibly downgraded for all users.
- Production financial values still originate from verified Go API/DB; local caches must not become independent authorities for balances or successful money mutations.

## 16. R3 job/worker deployment gate

Workers and queues are **future R3 scope unless earlier explicitly authorized**. No River queue, Redis, RabbitMQ, Kubernetes job controller or managed service is selected here.

| Risk | Release requirement |
|---|---|
| Durability | Persist job intent, owner, engine/model/baseline version, inputs reference and state transitions |
| Tenant isolation | Owner authorization at submit/status/result/cancel and inside worker data access |
| Resource exhaustion | Approved per-user concurrency, run limits, time/memory budgets, queue backpressure |
| Retry | Leases, idempotency/deduplication, poison job and timeout behavior demonstrated |
| Cancellation | Stop safely, no partial results presented as complete; return explicit status |
| Reproducibility | Deterministic version/seed/inputs where model supports replay |
| Privacy | No seed/input path, financial snapshot or raw user payload emitted to public logs |
| Upgrade | Compatible job payload/schema, drain old worker or safe migration and resumption |
| Results | Immutable result metadata, validity/staleness and retention consistent with user data rights |
| Operations | Real load, failure and recovery tests for worker/storage provider, with alerting |

A Monte Carlo run marked `COMPLETED` cannot be trusted if technically failed paths were silently discarded or a stochastic model remained unapproved. Objective/solver claims for optimizer require `Q-07` approval and independent validation.

## 17. Observability, diagnostics and alerting

Observe the system enough to troubleshoot failures **without turning diagnostics into a second copy of the financial database**.

| Signal | Safe example | Must not include |
|---|---|---|
| Availability | Overall readiness failures by environment/build | Tokens, customer names, private resource identifiers |
| Error | Sanitized error class and trace/correlation ID | Raw SQL with user data, request body, exception secrets |
| Latency | Handler/job timing histograms in buckets | User-specific balance/goal figures as labels |
| Database | Connection pool pressure, slow-query counts, lock time, migration revision | Query parameter values containing PII |
| Money mutation | Aggregate failure/reconciliation alarms, idempotency conflict rate | Actual transfer amounts or recipient personal data in open metrics |
| Worker | Queue depth, lease expiration, run outcomes, cancellations | Private scenario inputs and financial paths |
| Security | Repeated denied access, suspicious token validation failures | JWT contents or raw identity documents |
| Backup | Backup failure, age, restore drill status | Backup contents and encryption keys |
| Mobile | Crash/error events with opt-in/disclosure as required | Screen captures containing financial records |

A concrete SLO/error budget and paging thresholds must be **approved after measured load and owner risk review**. Alerts should route to a named responder, have escalation and runbooks, be tested, and not disclose private user details in push/email previews. Audit telemetry vendors and log retention according to `SECURITY-PRIVACY.md`.

## 18. Capacity, resilience and cost governance

**No performance target has been approved.** Determine capacity from observed workload and risk of actual-finance data loss, not from generic application tutorials. The planning process should record:

- Expected **concurrent users**, traffic mix, request size distribution, peak factor and geographical access patterns.
- Go API CPU/memory, connection-pool limits, slow-query thresholds and PostgreSQL I/O, CPU, locks, WAL and disk capacity.
- Mobile network conditions and compatibility windows; clients may retry after timeout or reconnect after days offline.
- Projected job queue depth, Monte Carlo runs × paths × horizon, per-tenant fair scheduling, cancellation, and resource isolation.
- Operational dependency limits: auth provider availability, DB failover, backup storage access, log sink capacity and credential rotation.
- Cost categories: compute, DB, backup/PITR, logs/traces, external auth, mobile signing/CI, egress, worker execution and on-call burden.

**Performance readiness** means a known workload plus test evidence against owner-approved thresholds. A recommended optimization cannot be declared sufficient without measurements and an operating cost/risk trade-off.

### 18.1 Failure-domain checks

| Fault injection / condition | Expected safe result | Evidence required |
|---|---|---|
| Temporary Go API loss | In-flight writes resolve to committed/failed/unknown with safe recovery; no duplicate postings | Request ID, idempotency replay, DB checks |
| DB connection exhaustion | Backpressure/error without public SQL/credential disclosure | Metrics, logs, denied-resource behavior |
| DB failover / network split | No double credit/debit, reconciliation possible, recovery runbook | Isolation and transactional tests |
| Identity provider unreachable | No silent bypass; clear client session/error recovery | Negative auth/consumer tests |
| CI/CD or signing credential leak | Revoke/rotate and investigate, pause release | Incident evidence and credential rotation record |
| Logging vendor outage | Privacy preserved and critical service behavior defined | Controlled fault test; no buffer of unencrypted sensitive payloads |
| Worker crash / lease expiry | Durable, deduplicated job recovery or explicit failure | Job state and replay tests |
| Stale scenario result after baseline mutation | Result marked stale / guarded by approved semantics | Version compatibility test |
| Mobile old app after backend update | Compatible API path or deliberate supported-version error | Version-skew smoke |
| Backup corrupt / missing | Alarm and incident path, no false “restorable” claim | Automated detection and separate restore rehearsal |

## 19. Privacy and personal-data lifecycle before launch

Production readiness includes **data governance**, not just deploying an API:

1. Classify the personal and financial data collected by each enabled feature, including scenario snapshots, exported files, job metadata, analytics and crash diagnostics.
2. Document purpose and lawful basis, user disclosure and appropriate consent/withdrawal mechanisms; obtain qualified legal/privacy review where needed. This file is **not** a legal opinion about Indonesian PDP/PSE obligations.
3. Approve retention schedule for source data, archived transactions, financial results, generated exports, logs, backups and incident evidence. Avoid invented values such as “keep all data for seven years” without review.
4. Protect user access/export and deletion workflows (`FR-021`) with server-side identity verification and ownership checks; test cross-tenant denial and export expiry.
5. Handle deletion and backup restore consistently: restoring an older backup must not silently reintroduce user data that was legitimately deleted. Define reconciliation/tombstone/purge policy before production.
6. Inventory third-party services, subprocessors, data regions/cross-border paths, admin/support access and analytics SDKs; decide which require disclosure, consent and contracts.
7. Use synthetic financial records for screenshots, support training, observability tests and CI. Scrub raw money amounts and identifying inputs from test recordings.
8. If a breach or privacy-impacting incident occurs, initiate the dedicated incident and regulatory notification assessment defined in Security & Privacy; do not claim a statutory deadline from this document alone.

**Hard stop:** don't enable external collection of personal financial information when core identity, ownership, retention, deletion, vendor-data-flow or incident responsibilities remain unreviewed.

## 20. Incident response, disaster recovery and communications

A single runbook should identify **incident commander**, technical owner, security/privacy contact and approver based on the real team's assigned responsibilities. Roles are placeholders until assigned.

### 20.1 Detection → contain → diagnose → recover → review

1. **Detect and verify:** collect time, environment, affected build/migration, alert ID and high-level impact. Preserve privacy-safe evidence.
2. **Classify severity:** separate data breach, incorrect financial ledger, inability to access app, lost backup, auth outage and model-accuracy incident. Reference proposed severity criteria in Security & Privacy; do not invent a formally approved on-call SLA.
3. **Contain:** reduce exposure, disable risky mutations, revoke compromised credentials and isolate affected services without deleting evidence; exact action needs authorized operator.
4. **Establish transaction truth:** compare committed DB state, idempotency records, audit trail and reconciliation totals. Client-visible timeout is not a reliable success/failure oracle.
5. **Choose recovery:** compatible code revert, forward-fix, account-safe domain correction, backup restore, job reprocessing or manual case handling under approval.
6. **Recover and verify:** re-run security/finance smoke, reconciling precise-money outcomes, tenant isolation and post-recovery deletion behavior.
7. **Notify appropriately:** route user/regulator communications through authorized owner/privacy/legal review with clear affected scope and factual timelines; avoid speculative assurance.
8. **Post-incident review:** causal timeline, root cause, data impact, what was/was not known, corrective tasks with IDs, test gaps and updated runbook.

### 20.2 Financial-specific escalation

Treat any cross-user access, duplicate posting, lost transfer leg, unexpected mutation of actual money by scenario, or corrupted net-worth recomputation as a potential **release-stopping incident**. Do not “repair” balances by an opaque `UPDATE` outside the approved domain correction and audit process.

## 21. Release readiness gates by phase

A completed documentation checklist is not sufficient; **mark only actual evidence** as passed.

| Gate | R0 | R1 | R2 | R3 |
|---|---|---|---|---|
| Repo/toolchain/build provenance | Required for applicable artifacts | Recheck | Recheck | Recheck |
| Flutter + Go tests / code quality | Relevant foundation tests | Full R1 regressions | R2 regressions + R1 | R3 regressions + R2 |
| API/DB contract and migration compatibility | Initial skeleton, if implemented | Mandatory for real finance | Mandatory + scenario schema | Mandatory + job/result evolution |
| Auth/tenant negative tests | Foundation auth if implemented | **Mandatory** before personal data | Extended to scenarios | Extended to jobs/results |
| Ledger/idempotency/reconciliation | Harness ready | **Mandatory** | Remain green | Remain green |
| Privacy/export/delete and retention | Plan and risk review | **Mandatory** before production | Scenario retention included | Worker/model retention included |
| Backup/restore/incident drill | Plan/design; selected staging drill | **Mandatory** before real-user production | Recheck changed DB/job paths | Recheck recovery under worker load |
| Deterministic reproducibility/staleness | Not applicable to R0 product | Not applicable unless feature shipped | **Mandatory** | Remain green |
| Optimization model validity | Not applicable | Not applicable | Not applicable | **Mandatory with Q-07 approval** |
| Monte Carlo statistical disclosures | Not applicable | Not applicable | Not applicable | **Mandatory with Q-08 approval** |
| Worker quotas/retry/cancel/isolation | Not applicable unless separately scoped | Not applicable | If any workers ship | **Mandatory** |
| Real-environment deploy authorization | No implied public deployment | **Explicit approval required** | Explicit scope/release approval | Explicit scope/release approval |

**R1 cannot pass while a real-money ledger invariant fails, even if all Flutter screens render.** R3 cannot pass with an unreviewed stochastic model or unbounded job execution. Use the universal `DOD-G01`–`DOD-G10` gates, release rows and `TS-*` test suites in their original documents.

## 22. Invariants for release engineering (`OPS-I01`–`OPS-I24`)

These are requirements for the eventual operations implementation, **not test results**.

| ID | Invariant / prohibited outcome |
|---|---|
| `OPS-I01` | A documentation ZIP or successful build never counts as deployed application evidence. |
| `OPS-I02` | An unresolved provider/host/migration choice is not implicitly approved by example YAML or commands. |
| `OPS-I03` | Production actions require explicit authorized approval and an identified environment. |
| `OPS-I04` | No production secrets, signing keys or personal financial records enter Git, agent prompts, test artifacts or logs. |
| `OPS-I05` | Local/CI/staging cannot silently operate on production financial data. |
| `OPS-I06` | All private API requests enforce server-side verified principal and resource ownership. |
| `OPS-I07` | App-owned balances and transfers are changed only by approved atomic/idempotent actual-finance operations. |
| `OPS-I08` | A client timeout cannot be treated as proof that a money write did not commit. |
| `OPS-I09` | A scenario preview, optimizer proposal or stochastic run never posts actual money. |
| `OPS-I10` | Schema/API version compatibility with existing supported mobile clients is reviewed before promotion. |
| `OPS-I11` | The deployment artifact is tied to an inspected commit, digest, build/dependency versions and approvals. |
| `OPS-I12` | Destructive schema changes and production database resets never run by default. |
| `OPS-I13` | A database restore always uses approved isolation and must be measured before claiming RPO/RTO compliance. |
| `OPS-I14` | Restoring backup data respects reviewed deletion/retention obligations and reconciles affected state. |
| `OPS-I15` | A backup schedule by itself never proves recovery readiness. |
| `OPS-I16` | Public network/diagnostic surfaces cannot expose PostgreSQL, secrets or private financial payloads. |
| `OPS-I17` | Client-side feature flags do not override backend authorization or model approval. |
| `OPS-I18` | Failed or unexecuted tests are not reported `PASS`, and documented features are not reported `DONE` without evidence. |
| `OPS-I19` | Job cancellation/crash/timeout cannot mark incomplete financial simulation results as complete. |
| `OPS-I20` | Model outputs carry approved model/baseline provenance; stale/invalid outputs cannot be silently reused. |
| `OPS-I21` | Shipping mobile apps requires protected platform signing and old-client compatibility planning. |
| `OPS-I22` | Production observability is useful without leaking user financial data or unrestricted vendor telemetry. |
| `OPS-I23` | Incident and rollback plans protect committed financial truth rather than editing ledger entries manually. |
| `OPS-I24` | An agent request to draft documentation never authorizes production release, account deletion or paid external services. |

## 23. Operational acceptance test catalog (`OPST-001`–`OPST-048`)

**Initial status of every test below: `NOT_RUN`.** All tests use synthetic data and a disposable approved environment unless a separate, narrowly scoped authorization permits an alternative. Attach test evidence to the canonical task and `PROGRESS.md` rather than claiming execution here.

### 23.1 Source, CI, artifact and configuration (OPST-001–OPST-012)

| ID | Scenario | Pass condition / evidence |
|---|---|---|
| `OPST-001` | Build revision is reproducible | Git commit, lockfile/toolchain version, artifact hash documented |
| `OPST-002` | Runner lacks production secrets on untrusted PR | Negative capability verified with redacted logs |
| `OPST-003` | Secret accidentally committed in synthetic fixture | Scanner blocks and produces no raw secret leak |
| `OPST-004` | Dependency with known critical issue | Approved policy blocks/reviews promotion; issue tracked |
| `OPST-005` | CI unit/widget/API suite fails | Mandatory stage blocks release and reports `FAIL` |
| `OPST-006` | Required suite is skipped | Skip cannot be misrepresented as `PASS` |
| `OPST-007` | Build has missing signing identity | Mobile release fails closed; signing key not exposed |
| `OPST-008` | Staging env points at prod DB credentials | Misconfiguration detected and blocks deployment |
| `OPST-009` | Wrong environment/client base URL | Production artifact cannot reach dev-only test endpoint |
| `OPST-010` | Config revision incompatible with approved runtime | Startup/readiness fails safely, operator alerted |
| `OPST-011` | Artifact tampering or digest mismatch | Deployment refuses artifact or records incident |
| `OPST-012` | Attempt release without owner approval | Production promotion blocked and audit event recorded |

### 23.2 API, identity, finance and PostgreSQL migration (OPST-013–OPST-024)

| ID | Scenario | Pass condition / evidence |
|---|---|---|
| `OPST-013` | Old mobile client calls new API | Supported contract succeeds or approved safe version error |
| `OPST-014` | Token invalid or expired | Access fails closed without private data disclosure |
| `OPST-015` | User A requests user B's nested resource | Cross-tenant denial on live-equivalent role and query |
| `OPST-016` | New schema migration applied to disposable DB | Idempotent runner behavior and effective schema recorded |
| `OPST-017` | Migration locking/timeout fault | No unexpected partial state, safe incident/rollback decision |
| `OPST-018` | Expand/contract version skew | Compatible client/server/schema matrix passes |
| `OPST-019` | Transfer request commits but response is lost | Retry/status check does not create duplicate transfer |
| `OPST-020` | Partial transfer failure injected | Debit and credit remain atomic/reconciled |
| `OPST-021` | Repeated income/expense mutation | Approved idempotency semantics prevent double posting |
| `OPST-022` | Actual money serialization round-trip | No precision loss through Flutter/Go/API/PostgreSQL |
| `OPST-023` | Restore/cutover changes source snapshots | Net worth and scenario staleness follow approved rules |
| `OPST-024` | DB role misconfiguration/RLS (if selected) | Privilege/tenant tests catch possible data exposure |

### 23.3 Backup, restore and incident controls (OPST-025–OPST-036)

| ID | Scenario | Pass condition / evidence |
|---|---|---|
| `OPST-025` | Scheduled backup fails | Alert reaches approved responder; no false green status |
| `OPST-026` | Backup restore to isolated environment | Integrity, schema, reconciliation and owner tests pass |
| `OPST-027` | Restore elapsed time measured | Compared with explicitly approved RTO, reported accurately |
| `OPST-028` | Recovery point inspected | Actual data loss window compared with approved RPO |
| `OPST-029` | Corrupt/incomplete backup | Reject, alert, try approved recovery candidate, preserve evidence |
| `OPST-030` | Deleted-user record appears in old backup | Approved deletion/tombstone reconciliation applied |
| `OPST-031` | Backup storage access requested by untrusted role | Access denied; audit of privileged access exists |
| `OPST-032` | TLS/secret rotation exercised | Service continuity and revocation demonstrated safely |
| `OPST-033` | Auth provider outage | Fail-closed access and clear error; no bypass |
| `OPST-034` | API/DB outage during financial write | Truth determined from DB/idempotency, not client timeout |
| `OPST-035` | Security incident drill | Named roles, escalation, evidence and containment documented |
| `OPST-036` | Revert code with incompatible DB schema | Preflight blocks unsafe rollback; forward-fix plan used |

### 23.4 Mobile, R2/R3 workers and privacy (OPST-037–OPST-048)

| ID | Scenario | Pass condition / evidence |
|---|---|---|
| `OPST-037` | Mobile app upgrade on supported device | Session, data, migration and navigation states remain safe |
| `OPST-038` | Old mobile app remains installed after API deploy | No incompatible mutation or hidden double posting |
| `OPST-039` | Flutter release built with debug instrumentation | Production gate detects and rejects unintended debug behavior |
| `OPST-040` | Analytics/crash event contains synthetic finance payload | Redaction/filter blocks sensitive transfer to vendor |
| `OPST-041` | Simulation job worker dies after lease | Approved retry/dedupe preserves job truth and result status |
| `OPST-042` | Cross-user simulation result requested | Go authorization denies access |
| `OPST-043` | R3 run exceeds quota | Backpressure/cancel/error is safe; no OOM cascade |
| `OPST-044` | R3 model/seed/baseline changes after result | Result provenance and staleness clear; no false replay claim |
| `OPST-045` | Export temporary URL leaked/expired | Scoped authorization/expiry protects personal export |
| `OPST-046` | User deletion from active and backup lifecycle | Reviewed retention and re-delete reconciliation proved |
| `OPST-047` | Startup readiness with missing required DB/auth dependency | Not ready; no false success health report |
| `OPST-048` | Full R1/R2/R3 release rehearsal | Applicable suite, runbooks, explicit approval and residual risks documented |

**Execution evidence record (per test):** test ID; date/time; commit/digest; environment and versions; synthetic fixture description; command/config reference (redacted); expected and observed behavior; exact `PASS/FAIL/BLOCKED/NOT_RUN/SKIPPED` status; log/report links; reviewer; associated task and issue ID.

## 24. Pre-deploy, deploy and aftercare checklists

### 24.1 Pre-deploy (copy into a real release ticket)

- [ ] Release and scope approved (phase/slice, affected `FR-*`/`EN-*`/`NFR-*`).
- [ ] The real target environment, owner, role access and data classification have been inspected.
- [ ] Applicable `Q-*`/`OPS-Q*` decisions show explicit recorded approval.
- [ ] Exact immutable artifact, commit, toolchain, config revision and API/schema contract identified.
- [ ] CI/security/financial/DB/client compatibility tests actually executed with evidence.
- [ ] Privacy and data-retention implications, export/delete, logging and third-party SDK review addressed.
- [ ] Migrations reviewed for locks, backfill, backward compatibility, idempotency and forward fix.
- [ ] Recent recovery capability and required restore drill verified; actual RPO/RTO compared to policy.
- [ ] Rollback/forward-fix path and incident contact exercised or reviewed as appropriate.
- [ ] Monitoring, redacted logs, health/readiness and high-severity alert routing verified.
- [ ] Production deployment and any destructive or paid action expressly authorized by the right human.

### 24.2 Controlled execution (operator-authorized only)

- [ ] Record start time and release baseline; freeze unrelated changes to affected deployment.
- [ ] Verify selected target and credentials **without printing their values**.
- [ ] Apply approved migration only through inspected migration process and safe order.
- [ ] Promote immutable reviewed backend/client artifacts using actual supported provider workflow.
- [ ] Run safe smoke checks with isolated synthetic identities and correct tenant boundaries.
- [ ] Verify idempotency, ledger truth, access denial, and appropriate mobile compatibility.
- [ ] Check error rate, DB pressure, worker queue (if deployed) and backup monitoring.
- [ ] Pause/contain and escalate if actual-money integrity, privacy or critical rollback issue emerges.
- [ ] Record final state `DEPLOYED` only after actual target confirmation and approval.

### 24.3 Post-deploy / handoff

- [ ] Record commit/tag, artifact digest, migration revision, environment and rollout completion time.
- [ ] Retain privacy-safe links to test/release/restore evidence; no raw exports or credentials.
- [ ] Update `PROGRESS.md` with specific task/epic IDs and observed status.
- [ ] Log open incidents, known limitations, decisions and follow-up owner.
- [ ] Reassess alert baseline during an approved monitoring window; do not invent a fixed period.
- [ ] Record mobile distribution status separately from backend deploy and app-store submission.
- [ ] Reconcile docs/contract changes via owner review; do not rewrite history of approvals.

## 25. Failure triage and release stop conditions

| Condition | Classification | Default action |
|---|---|---|
| Cross-user private data visible | Critical security/privacy | Contain and incident response; stop promotion |
| Atomic transfer failed, duplicated write, wrong amount | Critical financial integrity | Halt affected writes; reconcile via approved actual-finance process |
| Planned scenario applied to actual balance | Critical domain violation | Contain; investigate money ledger and restore/correction policy |
| App cannot verify identity / secret compromised | Critical auth exposure | Fail closed, revoke/rotate/contain; incident review |
| Migration causes incompatible schema on live clients | High/critical | Pause, safe compatible fix/rollback decision |
| Backup missing / failed restore drill | Blocker for public-data readiness | Do not declare production recovery readiness |
| Invalid stochastic probability or unapproved optimizer | R3 financial trust blocker | Keep feature unavailable; correct model/approval/testing |
| Noncritical visual regression in a low-risk screen | Task/release triage | Apply approved severity and scoped release criteria |
| Slow query/worker load beyond approved limits | Operational gate failure | Measure and remediate before promotion |
| Unknown test status or unverified service | `UNVERIFIED`/`NOT_RUN` | Do not convert to passed gate |

A planned demo may be possible using synthetic data and disabled high-risk behavior, with explicit disclosure. That is not permission to claim the financial product has passed real-user launch readiness.

## 26. Definition of Ready / Done for an operations task

### 26.1 Definition of Ready (before touching infrastructure)

- [ ] A task `TK-*`/`EP-*` or approved release ticket names the scope and target environment.
- [ ] Actual repository and infrastructure identifiers have been discovered; no guessed command/endpoint.
- [ ] Human owner, necessary decisions, credentials/permissions and approval identified.
- [ ] Risk, blast radius, expected downtime, migration, rollback/forward-fix and test plan documented.
- [ ] A nonproduction validation path exists; real users/production data are not used by default.
- [ ] Financial/security/privacy implications and source-of-truth docs checked.

### 26.2 Definition of Done (for implementation, not this document)

- [ ] Required code/config/manifests exist in verified repository and were peer reviewed.
- [ ] Applicable testing, security, migration and recovery suites executed with timestamped evidence.
- [ ] No critical open financial integrity, tenant isolation, secret, or unapproved model blocker.
- [ ] Environment verification and any production authorizations recorded.
- [ ] Deployment status reported accurately; released versions can be traced to immutable artifacts.
- [ ] `TASKS.md`, `PROGRESS.md` and the relevant Decision Log records synchronized with **observed** facts.
- [ ] Runbooks, service owners, access, backup/restore results and operator handoff are current.
- [ ] The global `DOD-G01`–`DOD-G10` gates and release-specific Testing Strategy gates are met.

For a **documentation-only task**, this file can be `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL` after structural, link and consistency checks, without any claim of production implementation.

## 27. Open deployment decisions (`OPS-Q01`–`OPS-Q18`)

The following **local subquestions** route to parent `Q-10` unless a row names another parent. Initial state of **every** local question is `TBD` / owner approval `NOT_RECORDED`. Do not create 18 separate product decisions by copying this table.

| ID | Proposed decision requiring approval | Parent / earliest impact |
|---|---|---|
| `OPS-Q01` | Hosting provider, cost/operating model, production regions, data-residency implications | `Q-10`, before public R1 |
| `OPS-Q02` | Environment topology (isolated accounts/projects, staging exposure, service count) | `Q-10`, R0 staging design |
| `OPS-Q03` | CI provider, branch protection, artifact registry, release approval/credentials | `Q-10`, R0 pipeline |
| `OPS-Q04` | Go API runtime, deployment method, probe/ingress topology | `Q-10`, before actual deploy |
| `OPS-Q05` | PostgreSQL provider/version, migration runner, app/migrator roles, RLS approach | `Q-03`, `Q-10`, R0/R1 |
| `OPS-Q06` | Monetary ledger schema and migration compatibility gates | `Q-03/Q-04`, before actual writes |
| `OPS-Q07` | Auth IdP/service keys, renewal/revocation, token verification, mobile clients | `Q-02`, before private production |
| `OPS-Q08` | DNS, TLS management, network access, public API, support/admin access | `Q-10`, before ingress |
| `OPS-Q09` | Approved RPO/RTO, backup/PITR schedule, encryption/key control, retention | `Q-10`, pre-public R1 |
| `OPS-Q10` | Frequency/scope of restore drills and disaster recovery ownership | `Q-10`, pre-public R1 |
| `OPS-Q11` | Incident severity, responders, escalation, customer/regulatory handling | `Q-10`, pre-public R1 |
| `OPS-Q12` | Log/metric/tracing vendor, safe event allowlist, sampling and retention | `Q-10`, R0/R1 |
| `OPS-Q13` | Supported mobile stores/platform versions, signing and release cadence | `Q-01/Q-10`, mobile launch |
| `OPS-Q14` | API compatibility window and old-client/deprecation policy | `Q-10`, pre-public R1 |
| `OPS-Q15` | Load/cost/SLO targets, traffic and worker budgets, availability monitoring | `Q-10`, before performance claims |
| `OPS-Q16` | R3 queue/worker provider, lease/retry/cancel semantics, quotas | `Q-07/Q-08/Q-10`, S-10/R3 |
| `OPS-Q17` | Data export/deletion retention, re-delete after restore and third-party processors | `Q-10`, `FR-021` pre-public R1 |
| `OPS-Q18` | Feature gates, controlled rollout/rollback strategy and final production sign-offs | `Q-10`, before live release |

**Approval record template** (maintain in `DECISION-LOG.md`, not as chat-only agreement): decision ID, options and trade-offs, chosen option, reviewer(s), effective version/slice, approval timestamp, evidence links, security/finance effect, rejected alternatives and follow-up task IDs.

## 28. Agent/operator execution workflow

A coding agent preparing operations changes follows [Development Workflow](../development/DEVELOPMENT-WORKFLOW.md): **Inspect → Trace → Plan → Implement → Verify → Handoff**.

For an ops task, the minimum deliverables are:

1. **Inventory:** verified code/config/provider evidence and current environment/permissions.
2. **Traceability:** task ID, `Q-*` and `OPS-Q*` impacts, release scope and source docs.
3. **Plan:** concrete files/resources, safe operation order, compatibility, tests, recovery and approvals.
4. **Changes:** smallest reviewed artifact/code changes; no production mutation without separate authorization.
5. **Verification:** actual command and safe redacted output, timestamps, environment, fixture, PASS/FAIL status.
6. **Handoff:** changed files, blocker list, decisions, next action, and no fake deployed status.

### 28.1 Handoff form (copy, do not pre-fill with imaginary success)

```text
OPERATIONS HANDOFF
Task / slice / release:       [TK-... / S-... / R...]
Requested scope:              [exact change]
Repository revision:          [verified commit or UNVERIFIED]
Target environment:           [local / CI / staging / prod + verified identifier]
Applied artifact / digest:    [actual, or NOT_APPLICABLE]
Schema revision:              [actual applied version, or NOT_VERIFIED]
Relevant decisions:           [Q-* / OPS-Q* with approval evidence]
Tests / safety checks:        [test ID, command, time, observed PASS|FAIL|NOT_RUN]
Backup / restore evidence:    [real report or NOT_RUN]
Owner approval:               [record / NOT_RECORDED]
Rollout state:                [NOT_STARTED|PARTIAL|BLOCKED|DEPLOYED|FAILED|ROLLED_BACK]
Financial/security impact:    [observed facts and remaining risks]
Changed files / resources:    [only inspected actual paths]
Monitoring / incident owner:  [real operator or NOT_ASSIGNED]
Follow-up tasks:              [exact IDs]
```

`DEPLOYED` is valid **only** when verified in the specific target environment. `ROLLED_BACK` applies to actual release actions only and does not imply that every schema/data change was reversed. Distinguish plan from action.

## 29. Documentation review and unverified items

### 29.1 Review performed for this documentation artifact

- Deployment architecture remains compatible with Flutter/Go/PostgreSQL/monorepo intent and the proposed modular-monolith arrangement.
- Production provider, region, secrets/CI vendor, deployment topology, backup/PITR/restore targets and retention remain `TBD` as required by `Q-10` and related Security & Privacy questions.
- No invented `docker compose`, `kubectl`, SQL migration, cloud CLI, app-store or signing command is presented as ready to run.
- Release gates inherit security, exact-money, idempotency, tenant isolation, contract and financial-simulation invariants.
- API and migration paths remain conditional on approved machine-readable contracts and actual checked-out files.
- Cross-references above target existing Markdown paths within the 37-document inventory.
- Test identifiers, local invariants, open decisions, and structural Markdown checks apply to **this document**, not the application.

### 29.2 What is deliberately **not** verified

- No Flutter/Go build, executable Go test, mobile store signing/upload, PostgreSQL migration/backup, CI workflow or cloud deployment executed.
- No production RPO, RTO, encryption at rest, incident contact, traffic budget, data retention or statutory compliance independently confirmed.
- No real cloud dashboard/host, secret store, health endpoint or deployed application observed.
- No actual feature has been moved from `UNVERIFIED` to `DONE` by creating this Markdown.

### 29.3 Mandatory next step after completing all 37 documentation artifacts

Conduct a **cross-document readiness audit** (without treating it as approval): inventory all 37 canonical files; check links, cross-references and version drift; reconcile decision registers; find unresolved R0 blockers; inspect the actual monorepo; then select the first authorized `S-00` task. The dated snapshots inside earlier Markdown files describe **their original writing time** and should be updated through explicit review rather than silently changing historical claims during packaging.

---

## Appendix A — Example conditional release ticket

The following is a **template, not an approved release**:

```yaml
release: <approved-release-id>
train: <R0|R1|R2|R3>
slice: <S-00...S-12>
source_revision: <verified-git-commit>
api_artifact_digest: <immutable-digest-or-NOT_APPLICABLE>
mobile_artifact_digest: <immutable-digest-or-NOT_APPLICABLE>
api_contract_revision: <approved-contract-id-or-TBD>
db_migration_revision: <verified-applied-revision-or-TBD>
environment: <verified-local|ci|staging|production>
approvals:
  owner: <approval-record-or-NOT_RECORDED>
  security_privacy: <approval-record-or-NOT_RECORDED>
tests:
  relevant_run: <evidence-reference-or-NOT_RUN>
backup_restore:
  most_recent_drill: <report-reference-or-NOT_RUN>
rollout:
  method: <reviewed-method-or-TBD>
  recovery_plan: <approved-runbook-reference-or-TBD>
state: <DRAFT|BLOCKED|STAGED|DEPLOYED|FAILED|ROLLED_BACK>
```

A YAML-like template is **not** a machine-readable config approved for CI/CD execution. Values between angle brackets denote placeholders; **never copy directly into a live release**.

## Appendix B — Example restore-drill evidence outline

```text
Drill ID:
Environment (verified nonproduction):
Operator and approver:
Backup source ID / hash (redacted):
Restore cut-off timestamp:
Start/end timestamps (with timezone):
Actual recovery point and actual elapsed recovery time:
Approved RPO/RTO reference (or TBD):
Database version / migration revision:
Integrity / row count / constraints checks:
Financial ledger reconciliation / sample golden cases:
Authz and cross-user negative tests:
Deletion / retention reconciliation:
Observed PASS/FAIL/BLOCKED (not pre-filled):
Private evidence reference:
Cleanup/temporary access revocation:
Corrective tasks:
```

## Appendix C — Referenced primary documentation

This playbook is built to be read with the canonical project sources. For implementation, start with [Documentation Index](../INDEX.md), [Decision Log](../planning/DECISION-LOG.md), [Testing Strategy](../development/TESTING-STRATEGY.md), [Definition of Done](../development/DEFINITION-OF-DONE.md), [Security & Privacy](../security/SECURITY-PRIVACY.md), [Database Design](../architecture/DATABASE-DESIGN.md), [API Guidelines](../architecture/API-GUIDELINES.md), and [Master Roadmap](../planning/MASTER-ROADMAP.md). External provider/cloud documentation may be added **only after** a provider is chosen and its version/feature set verified.

---

**Final documentation status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
**Deployment implementation status:** `UNVERIFIED`  
**Application test/deployment evidence:** `NOT_RUN`  
**Effect of completing the original documentation inventory:** this document is the **37th planned Markdown artifact**. `37/37` means documentation files have been prepared, **not** that all decisions are approved or Esokira is ready to deploy.
