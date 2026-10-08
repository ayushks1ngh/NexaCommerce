# NexaCommerce Implementation Roadmap

- **Document status:** PLANNED; Phase 0 is the current phase
- **Implementation status:** All application and infrastructure phases are NOT YET IMPLEMENTED
- **Last reviewed:** 2026-10-02

## 1. How to use this roadmap

Each phase follows **understand → design → implement → test → document/checkpoint** and must remain a small runnable increment. Do not begin the next phase merely because code compiles: explain the concepts, record tradeoffs, and satisfy the Definition of Done (DoD). Security, accessibility, documentation, and tests begin with the first relevant feature; their later named phases deepen and systematize them rather than postpone quality.

A Git checkpoint means a reviewed commit or tag candidate, not an automatic commit. Use focused commits and never include secrets. Phase ordering may be adjusted through a documented decision, but future infrastructure must not leak into an earlier phase without need.

## Phase 0 — Product and architecture foundation (CURRENT)

- **Objective:** Define the product, evolutionary architecture, contracts, data ownership, learning sequence, and initial technology decisions without implementation.
- **Concepts learned:** Requirements vs design, scope, quality attributes, service boundaries, tradeoff analysis, ADRs, living documentation.
- **Technologies involved:** Markdown, Mermaid, Git; technology evaluation only.
- **Implementation tasks:** Inspect repository; create PRD, architecture, roadmap, API/data baselines, curriculum, and four ADRs; label implemented vs planned; capture open decisions.
- **Expected deliverables:** The eleven documents under `docs/`; clean code-free repository state.
- **Tests:** Markdown structure/link checks where tooling exists; render Mermaid in a compatible viewer; manual checklist against Phase 0 requirements; inspect Git diff for accidental code/config.
- **Definition of Done:** Required sections/diagrams exist; terminology and ownership agree; proposals are not presented as deployed; no application/dependency/infrastructure changes; open questions are explicit.
- **Git checkpoint:** `docs: establish Phase 0 product and architecture foundation`

## Phase 1 — Next.js frontend foundation

- **Objective:** Build a small accessible web shell and public catalog mock experience without backend or authentication.
- **Concepts learned:** React component/state model, Next.js routing/layouts, server vs client components, rendering/caching basics, TypeScript boundaries, semantic HTML/CSS.
- **Technologies involved:** Node.js LTS chosen then, Next.js, React, TypeScript, styling/test tools selected explicitly.
- **Implementation tasks:** Create the app; define route/layout/error/loading structure; add typed fixture-based product list/detail UI; establish design tokens, accessibility baseline, environment/config policy, lint/type/test scripts; update README.
- **Expected deliverables:** Locally runnable responsive storefront shell using fixture data; no fake backend.
- **Tests:** Component tests for key states, route smoke tests, type/lint checks, keyboard/automated accessibility checks.
- **Definition of Done:** Fresh-clone instructions work; empty/loading/error states are visible; no secrets or Auth0 placeholders presented as real; critical UI is keyboard usable; decisions are documented.
- **Git checkpoint:** `feat(web): establish accessible Next.js storefront foundation`

## Phase 2 — Auth0 authentication

- **Objective:** Add secure login/logout/session handling and protected customer/admin route foundations.
- **Concepts learned:** OAuth 2.0, OIDC, authorization code + PKCE, cookies/sessions/tokens, issuer/audience/scopes, authentication vs authorization, CSRF/XSS/redirect threats.
- **Technologies involved:** Auth0 tenant/application/API and current supported Next.js integration.
- **Implementation tasks:** Threat-model browser flow; configure local tenant/application manually and document reproducible settings without secrets; implement login/callback/logout; protect routes; establish roles/permissions; display safe identity fields; define token/session handling.
- **Expected deliverables:** Authenticated web session and role-aware navigation using test users; configuration guide.
- **Tests:** Callback/state/nonce/session tests as supported; protected-route and role-negative tests; logout/expiry/failure smoke tests; secret/log review.
- **Definition of Done:** Hosted login succeeds; unauthorized access is denied server-side where a boundary exists; tokens are not placed in unsafe logs/storage; allowlists and environment separation are documented.
- **Git checkpoint:** `feat(auth): integrate Auth0 authentication foundation`

## Phase 3 — User Service

- **Objective:** Implement the first owned backend capability for profiles and addresses, linked to Auth0 identity.
- **Concepts learned:** Service structure, REST/OpenAPI, JWT validation, authorization, relational modeling, migrations, transactions, configuration, health checks.
- **Technologies involved:** TypeScript backend framework and ORM/query tool selected via spike/ADR, PostgreSQL, Auth0 JWKS.
- **Implementation tasks:** Decide framework/data tool; specify OpenAPI; create isolated service/database/migrations; map Auth0 `sub` to internal user idempotently; implement `/users/me` and address operations; add health/readiness and structured correlation logs.
- **Expected deliverables:** Independently runnable User Service, owned database, contract, migration/run docs, frontend profile/address integration.
- **Tests:** Unit validation/authorization; integration tests with real PostgreSQL; API contract and migration tests; cross-user/admin denial tests.
- **Definition of Done:** Only the service accesses its DB; Auth0 credentials are not stored; ownership is enforced; migrations work from empty DB; logs redact profile/token data; docs/ADR are updated.
- **Git checkpoint:** `feat(user): add profile and address service`

## Phase 4 — Product Service

- **Objective:** Replace catalog fixtures with an owned public catalog and minimal admin product/category management.
- **Concepts learned:** Catalog modeling, publication workflow, pagination/filter/sort, PostgreSQL text search/indexes, optimistic concurrency, cache measurement.
- **Technologies involved:** Existing backend stack, PostgreSQL, Next.js data fetching; no Redis by default.
- **Implementation tasks:** Define product/category schemas and contracts; implement published reads and permissioned admin writes; add basic PostgreSQL search/filter/sort; replace fixtures; define media references without premature media pipeline.
- **Expected deliverables:** Product Service/database, storefront data integration, minimal admin catalog UI.
- **Tests:** Domain and permission tests, query/integration/contract tests, pagination/search edge cases, admin/customer browser journey and accessibility checks.
- **Definition of Done:** Unpublished products cannot leak; admin mutations require permissions; query plans/indexes are reviewed with representative data; fixture fallback is removed intentionally; Redis is added only through ADR evidence.
- **Git checkpoint:** `feat(product): deliver catalog and product administration`

## Phase 5 — Cart Service

- **Objective:** Add a durable authenticated cart while keeping price and stock authority outside Cart.
- **Concepts learned:** Aggregate invariants, idempotent updates, optimistic concurrency, stale data, API composition, expiry/privacy.
- **Technologies involved:** TypeScript service stack, PostgreSQL, React/Next.js cart state.
- **Implementation tasks:** Define cart/item invariants; implement current-cart operations and quantity caps; show estimates from Product contract/projection; handle product changes; build cart UI and reconciliation warnings.
- **Expected deliverables:** Cart Service/database/API and complete add/update/remove/view journey.
- **Tests:** Quantity/invariant unit tests, DB/API contract tests, concurrent-update tests, cross-user isolation, stale/deleted product UI tests.
- **Definition of Done:** Cart is not price/stock authority; one user cannot access another cart; retries do not duplicate items unexpectedly; expiry/retention is documented.
- **Git checkpoint:** `feat(cart): add persistent customer carts`

## Phase 6 — Order Service

- **Objective:** Create durable order snapshots and a checkout state machine without real payment or distributed events.
- **Concepts learned:** Money/snapshots, state machines, idempotency keys, local transactions, orchestration, partial failure modeling.
- **Technologies involved:** Service stack and PostgreSQL; synchronous REST; stub/fake payment boundary.
- **Implementation tasks:** Resolve open currency/tax/shipping/address decisions; model order/items/status timeline; design checkout contract; validate product/cart; persist immutable snapshots; use an explicit fake payment adapter; add order history/detail/admin views.
- **Expected deliverables:** Order Service/database and end-to-end deterministic fake checkout.
- **Tests:** Totals/state-transition/idempotency unit tests; integration/contract tests; duplicate submission, stale price, failure and authorization journeys.
- **Definition of Done:** Same idempotency key cannot create duplicate orders; money avoids floating point; orders retain history; unknown/failure states do not show success; no inventory/payment service is simulated as complete.
- **Git checkpoint:** `feat(order): establish order and checkout workflow`

## Phase 7 — Inventory Service

- **Objective:** Add authoritative stock, reservations, adjustments, and oversell protection to checkout.
- **Concepts learned:** Concurrency control, locking/isolation, reservation expiry, compensation, audit ledgers.
- **Technologies involved:** Service stack, PostgreSQL, scheduled worker/process if justified.
- **Implementation tasks:** Model stock/ledger/reservation; define oversell and expiry policy; implement reserve/commit/release idempotently; integrate Order synchronously; add minimal admin inventory UI.
- **Expected deliverables:** Inventory Service/database and stock-aware checkout.
- **Tests:** Invariant/property tests; concurrent reservation integration tests; expiry/release/crash-recovery cases; permission and audit tests.
- **Definition of Done:** Demonstrated concurrent requests cannot oversell under the defined policy; every adjustment is attributable; retries and expiry are safe; recovery procedure is documented.
- **Git checkpoint:** `feat(inventory): enforce stock reservations and adjustments`

## Phase 8 — Payment Service

- **Objective:** Integrate one sandbox payment provider behind an isolated, secure payment boundary.
- **Concepts learned:** Provider adapters, tokenization/hosted payment UI, webhook signatures, idempotency, asynchronous outcomes, reconciliation, PCI scope awareness.
- **Technologies involved:** Payment provider SDK/API selected then, Payment Service, PostgreSQL, secrets handling.
- **Implementation tasks:** Compare/select provider; threat model; implement attempts/provider references; integrate hosted/tokenized UI; verify/deduplicate webhooks; reconcile timeout/pending states; connect to order/inventory compensation; create sandbox runbook.
- **Expected deliverables:** Sandbox purchase path with Payment Service/database; no raw card storage.
- **Tests:** Adapter tests, provider sandbox cases, webhook signature/replay/order tests, timeouts/declines/duplicates, end-to-end checkout and log/secret review.
- **Definition of Done:** Raw PAN/CVV never reaches storage/logs; duplicate requests/webhooks are safe; order/payment/inventory reconcile after known failures; provider outage behavior is visible and recoverable.
- **Git checkpoint:** `feat(payment): integrate secure sandbox payments`

## Phase 9 — Event-driven architecture

- **Objective:** Introduce asynchronous communication for a concrete non-blocking/reliability need, initially notifications and operational projections.
- **Concepts learned:** Broker semantics, at-least-once delivery, outbox, schema evolution, idempotent consumers, ordering, retries/dead letters, eventual consistency.
- **Technologies involved:** Broker chosen by ADR (RabbitMQ default proposal), outbox relay, notification consumer, OpenTelemetry context.
- **Implementation tasks:** Record broker ADR; define event envelope/catalog; implement transactional outbox; publish order/payment facts; build idempotent notification/log sink; define retry/DLQ/replay ownership; remove synchronous coupling only where justified.
- **Expected deliverables:** Durable event path and one useful consumer with operational docs.
- **Tests:** Producer/consumer contract tests; duplicate, delayed, reordered, broker outage, retry/DLQ and replay tests; outbox crash-boundary integration tests.
- **Definition of Done:** No dual-write data loss in tested boundaries; consumers are idempotent; schemas/versioning/PII are documented; DLQ is observable/actionable; checkout correctness does not rely on best-effort messaging.
- **Git checkpoint:** `feat(events): add reliable domain event delivery`

## Phase 10 — API Gateway

- **Objective:** Establish a single public API boundary when multiple APIs justify it.
- **Concepts learned:** Edge routing, BFF/API gateway tradeoffs, coarse auth, rate limits, CORS, aggregation, correlation, avoiding gateway business logic.
- **Technologies involved:** Thin Next.js BFF, reverse proxy, or managed/self-hosted gateway selected via ADR.
- **Implementation tasks:** Compare options; define public/internal routes; centralize TLS-facing routing, request IDs, limits and coarse token checks; keep service authorization; hide internal services; document timeout/payload policy.
- **Expected deliverables:** Versioned `/api/v1` public edge and migration plan from direct calls.
- **Tests:** Routing/contract tests, malformed/oversized/rate-limit behavior, auth bypass negatives, timeout propagation and trace correlation.
- **Definition of Done:** Only intended APIs are public; gateway contains no domain source of truth; defense-in-depth authorization remains; failure behavior is documented.
- **Git checkpoint:** `feat(edge): introduce governed public API gateway`

## Phase 11 — Docker and containerization

- **Objective:** Package understood applications reproducibly for local and later deployment.
- **Concepts learned:** Images/layers, build context, multi-stage/non-root builds, networking, health signals, volumes, supply-chain scanning.
- **Technologies involved:** Docker/Compose and image scanner selected then.
- **Implementation tasks:** Add per-component Dockerfiles; pin bases by version/digest policy; create local Compose dependencies; use non-root/read-only where feasible; keep secrets external; document build/run/debug.
- **Expected deliverables:** Reproducible local multi-service environment and deployable images.
- **Tests:** Image builds, container smoke/health tests, Compose integration journey, vulnerability/secret/image-size checks.
- **Definition of Done:** Fresh environment starts from documented commands; images contain no secrets/dev-only bloat; state persists only in declared volumes/services; shutdown/migration behavior is clear.
- **Git checkpoint:** `build: containerize NexaCommerce services`

## Phase 12 — Testing strategy expansion

- **Objective:** Turn existing phase tests into a balanced, reliable system-level strategy; not begin testing for the first time.
- **Concepts learned:** Test pyramid, contract/component/E2E/load/security/accessibility testing, test data, determinism, CI economics.
- **Technologies involved:** Existing test runners plus API, browser, contract, load, and accessibility tools selected explicitly.
- **Implementation tasks:** Document matrix/ownership; close critical coverage gaps; add consumer/provider contracts; automate critical purchase/admin journeys; build isolated fixtures; establish baseline performance and basic security/accessibility suites.
- **Expected deliverables:** Fast default suite, deeper staged suites, coverage/risk map, failure diagnostics.
- **Tests:** Meta-validation through seeded failures/flaky-test review; run unit, integration, contract, E2E, accessibility, baseline load, and recovery scenarios.
- **Definition of Done:** Critical risks map to tests; suites are repeatable and appropriately isolated; no arbitrary coverage percentage substitutes for risk; failure output is actionable.
- **Git checkpoint:** `test: establish cross-service quality strategy`

## Phase 13 — CI/CD

- **Objective:** Automate trusted validation, artifact production, and controlled environment delivery.
- **Concepts learned:** Pipeline stages, caching, least-privilege OIDC, artifact provenance, migration/deployment strategy, approvals/rollback.
- **Technologies involved:** GitHub Actions (default proposal), registry, scanners, deployment interface.
- **Implementation tasks:** Add PR quality/security workflows; pin actions; build/sign/version artifacts; generate SBOM as appropriate; use environment approvals and short-lived cloud credentials; design migration and rollback gates; protect branches.
- **Expected deliverables:** Reproducible CI and staged CD pipeline with documented controls.
- **Tests:** Workflow lint/smoke, intentionally failing checks, artifact verification, staging deployment/rollback and migration compatibility tests.
- **Definition of Done:** PRs cannot bypass required checks under normal policy; no long-lived cloud secrets in CI; same artifact is promoted; failed deployment can be detected and reversed/forward-fixed.
- **Git checkpoint:** `ci: add secure validation and delivery pipelines`

## Phase 14 — Cloud deployment

- **Objective:** Deploy a non-production environment first, then a production-shaped environment only after readiness review.
- **Concepts learned:** Provider/IaC choice, networks/IAM/KMS, managed compute/data, DNS/TLS, cost, scaling, backup/restore, environment isolation.
- **Technologies involved:** Selected cloud and IaC tool, managed container runtime, PostgreSQL, optional broker/Redis, secret manager.
- **Implementation tasks:** Record provider/compute/IaC ADRs; create least-privilege networked environments; deploy managed dependencies; configure DNS/TLS/secrets/backups/budgets; execute restore; write operations/decommission instructions.
- **Expected deliverables:** Reproducible staging and approved production architecture; inventory/cost and recovery records.
- **Tests:** IaC checks, post-deploy smoke/E2E, network/IAM negatives, backup restore, scaling/failure and teardown tests in safe environments.
- **Definition of Done:** No public database/cache/broker; environments/credentials separated; TLS and backups verified; spend alerts exist; deployment and restoration are reproducible; real customer launch remains separately approved.
- **Git checkpoint:** `infra: provision governed cloud environments`

## Phase 15 — Observability

- **Objective:** Make customer-impacting failure diagnosable and actionable across web, services, data, providers, and events.
- **Concepts learned:** Structured logs, metrics, distributed traces, OpenTelemetry, SLIs/SLOs/error budgets, alerts, dashboards, runbooks, telemetry privacy/cost.
- **Technologies involved:** OpenTelemetry and selected managed/cloud or Grafana-compatible backend.
- **Implementation tasks:** Propagate trace/correlation context; define critical-path SLIs; baseline and approve SLOs; add dashboards and symptom alerts; instrument broker/provider/DB; redact/sample/retain safely; create runbooks and synthetic checks.
- **Expected deliverables:** Purchase-path dashboards/traces/alerts and incident response documentation.
- **Tests:** Telemetry assertions, trace propagation, alert firing/routing, synthetic checkout, seeded dependency failure, PII/redaction review.
- **Definition of Done:** Operator can detect and diagnose representative failures without exposing sensitive data; alerts are actionable; SLOs are measured rather than invented; telemetry cost/retention is bounded.
- **Git checkpoint:** `ops: add production-oriented observability`

## Phase 16 — Production hardening

- **Objective:** Evaluate readiness for limited real-user launch and close high-risk security, reliability, privacy, performance, and operational gaps.
- **Concepts learned:** Threat modeling, abuse controls, capacity/error budgets, disaster recovery, incident response, privacy lifecycle, release/rollback, resilience exercises.
- **Technologies involved:** Existing stack plus approved scanning/testing/operations tools; no default new platform.
- **Implementation tasks:** Refresh threat models; resolve high risks; harden IAM/headers/rate limits/secrets/dependencies; validate retention/deletion; run load/capacity and accessibility review; exercise provider/broker/DB failure, restore, rollback, key rotation, and incident response; complete launch checklist.
- **Expected deliverables:** Evidence-based readiness report, accepted residual-risk register, recovery/security/privacy runbooks, launch or no-launch decision.
- **Tests:** Penetration/security review appropriate to scope, load/soak, restore and disaster exercise, failover/degradation, accessibility manual audit, end-to-end rollback and incident drill.
- **Definition of Done:** Owners explicitly accept residual risks; critical findings are closed; measured objectives and capacity are documented; support/on-call/incident/privacy processes exist; launch is reversible. “Production-ready” is not claimed solely because phases are complete.
- **Git checkpoint:** `chore: complete production readiness review`

## 2. Cross-phase exit questions

At every checkpoint ask: Can the developer explain the design without generated code? Is the smallest useful behavior implemented? Are negative/failure cases tested? Are security/data boundaries preserved? Did docs and ADRs change with reality? Can the increment run from a clean environment? Is the next technology solving a demonstrated problem? If not, remain in the current phase.
