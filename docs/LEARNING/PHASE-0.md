# Phase 0 Learning Record — Product and Architecture Foundation

- **Phase status:** COMPLETE (content authored); Git checkpoint PENDING
- **Implementation status:** Documentation only — NO application or infrastructure code exists
- **Last reviewed:** 2026-10-08
- **Related curriculum:** `docs/LEARNING.md` (overall syllabus)

This record documents what Phase 0 actually produced and what the developer should now be able to
explain. Phase 0 is a **thinking and documentation** phase. It deliberately contains no runnable
code, no dependencies, and no infrastructure. The claims below are grounded in the documents under
`docs/`; nothing here asserts learning or implementation beyond what those documents support.

---

## 1. Phase objective

Define the product, the evolutionary architecture, the contracts, the data ownership model, the
learning sequence, and the initial technology decisions — **without implementation**. Establish a
clean, code-free foundation that later phases build on one small increment at a time.

Learning objective: be able to reason about *what* is being built and *why the architecture is
shaped the way it is*, before writing any code.

---

## 2. Product requirements (from `docs/PRD.md`)

NexaCommerce is a learning-focused, production-oriented commerce platform. Customers discover
products, purchase them through a sandbox payment flow, and track orders; administrators maintain
catalog, inventory, orders, and user status.

- **Core customer journey (MVP):** discover → cart → checkout → sandbox payment → order tracking.
- **Admin journey (MVP, minimal):** manage catalog, stock adjustments, order transitions, user
  lookup/suspension, and basic operational visibility.
- **Explicit non-goals (MVP):** marketplace/multi-seller, subscriptions, recommendations, loyalty,
  native apps, multi-region, and introducing a broker/gateway/cache/containers/Kubernetes before a
  need is demonstrated.
- "Production-oriented" is **design intent**, not a readiness claim. Production readiness is a
  separate, explicit review (Roadmap Phase 16).
- Many commerce specifics (currency, tax, shipping, refund policy, reservation timeout, payment
  provider) are **open product decisions** recorded for later.

---

## 3. Architecture thinking (from `docs/ARCHITECTURE.md`)

The guiding idea is **evolutionary, coarse-grained services**, not one service per entity. The
repository currently contains only planning documentation and empty placeholder directories; there
is no frontend, API, service, database, cache, broker, container, pipeline, or cloud resource.

Status vocabulary is used consistently throughout the docs and must be respected:

- **DECIDED** — accepted direction (here or in an ADR).
- **PLANNED** — intended roadmap work, not yet delivered.
- **PROPOSED** — a recommended default awaiting phase-specific validation.
- **NOT YET IMPLEMENTED** — no running component should be inferred.

The initial implementation direction is sequenced: a Next.js + TypeScript frontend first, then
Auth0, then one backend capability at a time, with PostgreSQL-backed functionality before any
Redis, broker, gateway, or container platform.

---

## 4. Why microservices are being used (ADR-001)

Microservices are a **learning and design objective**, adopted *evolutionarily*, not eagerly.

- The domains (catalog, cart, order, inventory, payment) have distinct responsibilities and failure
  modes, which makes them a realistic vehicle for learning service boundaries, data ownership,
  REST, events, and distributed-system operations.
- But the project starts with one developer, no users, and no measured scale. Splitting into a
  service per entity up front would multiply deployments, network failure modes, and scaffolding
  before boundaries are understood.
- Therefore: implement **one coarse-grained owner at a time**, prohibit cross-service database
  access, and justify every split or merge of a boundary with evidence (recorded via ADR). A
  modular approach that later receives an isolated deployment/database is acceptable as long as
  **ownership is always unambiguous**.

---

## 5. Service ownership (from `docs/ARCHITECTURE.md` §4.4 and `docs/DATABASE.md` §2)

Each capability owns its business data and invariants; others use its API or events, never its
tables.

| Owner | Owns | Does **not** own |
|---|---|---|
| Auth0 | Credentials, identity-provider records, authentication factors | Commerce profile, addresses, orders |
| User Service | Application profile, Auth0 `sub` mapping, addresses, application status | Passwords, order records |
| Product Service | Products, categories, publication, preliminary reviews | Stock counts, carts, orders |
| Cart Service | Active carts and items | Authoritative product price/stock |
| Order Service | Order snapshots, order-item snapshots, checkout state | Product master, inventory ledger, payment truth |
| Inventory Service | On-hand/reserved quantities, reservations, adjustment ledger | Product descriptions, orders |
| Payment Service | Payment attempts/status, provider references, processed webhook IDs | Raw card data, order line items |
| Notification | Delivery preferences/status (if needed) | User identity, order truth |

The User Service is the **first** backend service. Auth0 authenticates; NexaCommerce services
authorize. The immutable Auth0 subject (`sub`) maps to an internal user; email is explicitly **not**
the durable cross-service identifier.

---

## 6. The Auth0 decision (ADR-002)

- **Decision:** Use Auth0 as the identity provider for customers and administrators. ACCEPTED;
  integration is PLANNED and NOT YET IMPLEMENTED.
- **Why:** Identity is security-critical but is not the commerce problem the project aims to
  reinvent. Outsourcing registration, login, logout, recovery, MFA, and token issuance to a
  specialist reduces risk, while NexaCommerce retains ownership of commerce profile data and
  authorization rules.
- **Boundary:** Auth0 owns credentials and core identity claims; the User Service owns the
  application profile and addresses and maps the Auth0 `sub` to an internal user. UI route guards
  improve experience but are **never** an authorization boundary — services validate tokens
  (signature via JWKS, issuer, audience, expiry, algorithm, permissions) server-side.

---

## 7. The PostgreSQL decision (ADR-003)

- **Decision:** PostgreSQL is the primary durable system of record. ACCEPTED; schemas are NOT YET
  IMPLEMENTED.
- **Why:** Commerce data (products, customers, carts, orders, stock, payment metadata) needs
  constraints, transactions, auditable state transitions, indexing, migrations, backups, and
  operational discipline — exactly what a mature relational database teaches and provides.
- **Ownership model:** "Database per service" means **logical** ownership with isolated
  credentials. During early local learning, separate databases may share one PostgreSQL
  server/cluster to control cost; separate physical clusters are an operational choice, not a
  prerequisite for ownership. Cross-service references are opaque IDs, never cross-database foreign
  keys.

---

## 8. The Redis decision and its conditions (ADR-004)

- **Decision:** Redis is ACCEPTED WITH CONDITIONS as an allowed supporting technology; adoption is
  PLANNED only when a concrete use case is demonstrated; NOT YET IMPLEMENTED.
- **Conditions:** Start **without** Redis. Measure browser/CDN/application/database behavior first.
  Redis must never become a source of truth for orders, payments, inventory, or user profiles.
- **If introduced**, every use must define: source of truth, key/version schema, TTL, invalidation,
  privacy, stampede protection, outage fallback, and hit/miss/latency/eviction signals. Likely
  earliest candidate is bounded catalog caching or rate-limit counters — only after measurement.

---

## 9. API principles (from `docs/API.md`)

- Model business resources and workflows, not database tables. JSON over HTTPS with clear REST
  semantics initially; no GraphQL planned.
- Validate at trust boundaries; return machine-readable, safe errors using `application/problem+json`
  (RFC 9457) with a stable `code`, human-safe `detail`, and a `traceId` that reveals no topology.
- Correct status-code usage (`2xx` success family; `400/401/403/404/409/412/422/429/5xx` errors);
  never return `200` for an error or leak stack traces/secrets/internal hostnames.
- Idempotency (via `Idempotency-Key`) is required for checkout and payment-affecting writes;
  optimistic concurrency (ETags/`If-Match`) protects contested admin updates.
- Paginate collections (cursor pagination as the default), constrain expensive queries, and use
  explicit allowlists for filter/sort fields.
- Public version boundary is `/api/v1` once a gateway/BFF exists; service-local development may use
  `/v1`. Internal service endpoints are not publicly routable in the target architecture.
- OpenAPI specs and consumer contract checks are deliverables of **service implementation phases**,
  not Phase 0.

---

## 10. Data ownership principles (from `docs/DATABASE.md`)

- A service is the authority for its data; others use its API/events, never its tables.
- No joins, foreign keys, views, triggers, shared ORM models, or shared credentials across service
  boundaries. Composite screens are assembled via API composition or purpose-built read models.
- Local transactions update only the owning service's records; cross-service consistency is
  explicit and generally eventual.
- Orders store **immutable snapshots** (product name, unit price, currency, quantity, address,
  totals) so history stays accurate even when the catalog changes.
- Migrations are owned per service, reviewed even when ORM-generated, tested on realistic data, and
  follow expand/migrate/contract; already-applied migrations are never edited (append a corrective
  one).
- Retention/privacy classification and RPO/RTO are open decisions to settle **before** real data.

---

## 11. Synchronous vs asynchronous communication (from `docs/ARCHITECTURE.md` §5)

- **Synchronous REST** is used when the caller needs an immediate answer to proceed: profile
  access, catalog reads, cart operations, and selected checkout commands. Calls require timeouts,
  cancellation, bounded retries only for safe transient operations, correlation context, and
  explicit error mapping. Avoid long synchronous call chains.
- **Asynchronous/event-driven** communication is introduced **later (Phase 9)** for facts that need
  not block the originating request (notifications, projections) and for resilient workflow
  propagation. Events are past-tense facts with owner, schema/version, event ID, aggregate
  ID/version, time, and trace context. Delivery is assumed **at least once**: consumers deduplicate
  and tolerate reordering. A **transactional outbox** prevents a committed DB change from losing its
  event; dead letters require an owned diagnosis/replay process.
- Checkout is a **workflow with compensation/reconciliation**, not a distributed ACID transaction;
  two-phase commit is not planned. Unknown outcomes stay **pending** and are reconciled — never
  fabricated as success.

---

## 12. Security principles (from `docs/ARCHITECTURE.md` §8 and `docs/PRD.md` §10.1)

- Defined trust boundaries: browser/edge, identity, public API edge, service, data, provider,
  admin, and telemetry/supply chain.
- Validate JWTs server-side (signature via JWKS, issuer, audience, expiry, algorithm, permissions);
  never trust UI role checks or browser-supplied claims alone.
- Least privilege everywhere; validate inputs and encode outputs; parameterized persistence.
- No raw passwords or card data stored; verify signed payment webhooks and prevent replay.
- Keep secrets out of source control, logs, browser bundles, and images; plan rotation.
- Protect browser flows against CSRF, XSS, clickjacking, session, and redirect attacks; narrow
  CORS. Do not log tokens, secrets, full addresses, or payment data.
- Threat modeling and a privacy/retention review are required **before** handling real
  customer/payment data.

---

## 13. Observability principles (from `docs/ARCHITECTURE.md` §9)

Observability is adopted **progressively**:

- Early phases: structured logs, request IDs, local errors, health endpoints.
- Multiple services: propagated trace context, RED metrics (rate/errors/duration), dependency/DB
  metrics, dashboards for critical paths.
- Events: publish/consume failures, lag, retries, dead letters, correlation.
- Production: OpenTelemetry instrumentation, centralized logs/metrics/traces, SLOs from measured
  needs, symptom-based alerts, runbooks, synthetic checkout, and tested backup restoration.
- Telemetry is itself access-controlled, retained data; never log tokens, secrets, raw payment
  data, or unnecessary personal data.

---

## 14. Deployment evolution (from `docs/ARCHITECTURE.md` §10)

- Local processes and managed emulators/sandboxes come first.
- Docker arrives only **after** services run and are understood (Roadmap Phase 11).
- CI/CD, cloud resources, and production topology are later roadmap outcomes.
- Kubernetes is **not** selected; a managed container application platform is preferred initially.
- Target properties include separate dev/staging/prod config and credentials, immutable artifacts,
  private data services, least-privilege identities, encrypted traffic, managed backups, and
  health-based rollout/rollback. Multi-region and service mesh are not planned without evidence.

---

## 15. ADRs created in Phase 0

| ADR | Title | Status |
|---|---|---|
| ADR-001 | Evolutionary microservices architecture | ACCEPTED as target direction; extraction PLANNED, NOT YET IMPLEMENTED |
| ADR-002 | Auth0 for customer and administrator identity | ACCEPTED; integration PLANNED, NOT YET IMPLEMENTED |
| ADR-003 | PostgreSQL as the primary durable datastore | ACCEPTED; schemas NOT YET IMPLEMENTED |
| ADR-004 | Redis for bounded ephemeral and caching use cases | ACCEPTED WITH CONDITIONS; adoption PLANNED only when demonstrated, NOT YET IMPLEMENTED |

---

## 16. Important architectural decisions (settled in Phase 0)

- Evolutionary coarse-grained services, implemented one owner at a time (ADR-001).
- Auth0 authenticates; NexaCommerce services authorize (ADR-002).
- PostgreSQL is the durable system of record; logical database-per-service ownership (ADR-003).
- Redis only when measured need exists, never authoritative (ADR-004).
- REST/JSON first, `/api/v1` public boundary, idempotency for checkout/payment, RFC 9457 errors.
- Order snapshots and compensation-based checkout instead of distributed transactions.
- Progressive security, observability, and deployment maturity rather than up-front platform.

---

## 17. Deferred decisions (recorded, not yet made)

From `docs/ARCHITECTURE.md` §11, `docs/PRD.md` §15, and `docs/DATABASE.md` §11:

- Node.js backend framework (Fastify proposed; validate via Phase 3 spike).
- ORM/database access tool (Drizzle vs Prisma; validate via spike).
- Message broker (RabbitMQ proposed default; Phase 9).
- API gateway/BFF approach (Phase 10).
- Cloud provider and IaC tool (Phase 14).
- Observability backend (progressive, OpenTelemetry-based).
- Target market, currency, tax model, shipping options, address validation, refund/cancellation
  policy, inventory reservation timeout, payment sandbox provider.
- Money representation details, ID format, RPO/RTO, retention durations, privacy jurisdiction.

---

## 18. Things intentionally NOT implemented in Phase 0

Everything executable. Specifically: no `apps/web`, no Next.js/React/TypeScript app, no Auth0
integration, no backend services, no PostgreSQL/Redis/broker/gateway, no Docker/CI/CD/cloud, no
catalog/cart/order/inventory/payment code, no fake backend, and no dependencies. The directories
`apps/`, `services/`, `infrastructure/`, and `.github/` exist as empty placeholders only.

---

## 19. Questions the developer should now be able to answer

- Why does this project adopt microservices *evolutionarily* rather than one service per entity?
- What does "database per service" mean here, and why can services share one PostgreSQL cluster
  while still owning their data?
- Where is the boundary between Auth0 and the User Service? Why is `sub`, not email, the durable
  identifier?
- Why does authentication (Auth0) not decide resource ownership, and where is authorization
  enforced?
- When is synchronous REST appropriate, and why is checkout a workflow with compensation rather
  than a distributed transaction?
- Why must checkout/payment writes be idempotent, and what does "at least once" delivery imply for
  consumers?
- Why is Redis not introduced yet, and what conditions must hold before it is?
- What is authoritative versus a snapshot/projection/cache in this architecture?
- Why does Docker/CI/CD/cloud come late in the roadmap?
- What is still an open decision, and what triggers each one?

---

## 20. Common mistakes to avoid (reinforced by Phase 0)

- Treating planning documents as if components already exist (respect the status vocabulary).
- Introducing a broker, cache, gateway, or containers before a demonstrated need.
- Letting one service read another service's database.
- Trusting UI role checks or browser-supplied claims as an authorization boundary.
- Making email the cross-service identity key.
- Storing raw card data or logging tokens/secrets/personal data.
- Claiming "production-ready" because phases are complete.

---

## 21. Phase 0 completion criteria (from `docs/ROADMAP.md`)

- Required sections and diagrams exist across PRD, Architecture, Roadmap, API, Database, Learning,
  and the four ADRs. ✔ Present.
- Terminology and ownership agree across documents. ✔ Verified during review.
- Proposals are not presented as deployed. ✔ Status vocabulary used consistently.
- No application/dependency/infrastructure changes. ✔ `apps/`, `services/`, `infrastructure/` empty.
- Open questions are explicit. ✔ Captured in deferred-decisions sections.
- Git checkpoint `docs: establish Phase 0 ...`. ◻ **Pending** — docs are authored but untracked.

---

## 22. Git checkpoint status

**PENDING.** As of the last review, all Phase 0 documentation is authored but the `docs/` tree is
untracked in git. Phase 0 content is complete; the phase is not fully closed until a reviewed,
human-approved commit exists. No commit or push is performed automatically.
