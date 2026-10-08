# NexaCommerce Project Learning Curriculum

- **Document status:** PLANNED and living
- **Learning model:** Understand → design → implement → test → deploy → explain
- **Last reviewed:** 2026-10-02

## 1. Using the curriculum

Learning is demonstrated by a working project decision or behavior, not by merely adding a dependency. For each topic, first explain the concept and failure modes, then design a narrow use, implement it in its roadmap phase, test positive and negative behavior, and update documentation. Keep short phase notes covering: what changed, why, one failure investigated, evidence from tests/telemetry, and what should be reconsidered.

| Topic | What to understand | Why it matters in NexaCommerce | When | Practical demonstration |
|---|---|---|---|---|
| **Git** | Commits, branches, diffs, ignore rules, merge/rebase concepts, tags, hooks, recovery and secret-safe history | Every phase needs reviewable change history and safe checkpoints | Phase 0 onward; CI policy in 13 | Commit only coherent phase work, review diffs, recover a non-destructive mistake, use protected PR checks later |
| **Web fundamentals** | Browser/server model, URLs, HTML semantics, CSS layout, forms, cookies, origins, CORS, caching, progressive enhancement and accessibility | Storefront, checkout, sessions and admin UI all cross browser trust/accessibility boundaries | Phase 1; reinforced 2, 10, 16 | Build semantic responsive catalog/forms; inspect browser requests/cookies/cache; complete critical flow by keyboard |
| **HTTP** | Methods, safety/idempotency, status codes, headers, content negotiation, caching validators, TLS, timeouts, retries and proxies | REST contracts, checkout retries, cache correctness and gateway behavior depend on exact semantics | Phase 1 basics; 3–10 applied | Implement API errors/statuses, ETags/idempotency, bounded timeouts; use network tools to explain request/response and retry safety |
| **TypeScript** | Structural typing, narrowing, generics, modules, strictness, runtime-vs-static validation, safe error modeling | Shared language across web/services reduces mistakes but cannot validate untrusted network/data input alone | Phase 1 onward | Enable strict checks; model UI/API states; validate runtime payloads; demonstrate a compile-time error and a runtime validation failure |
| **React** | Components, props/state, rendering, hooks, controlled forms, effects, composition, error/loading boundaries and accessibility | Customer/admin interactions require predictable state without unnecessary global complexity | Phase 1; expanded per feature | Implement product/cart/checkout states; test interaction/error behavior; explain why state is local, server-derived, or URL-derived |
| **Next.js** | App routing/layouts, server/client components, rendering/caching/revalidation, route handlers/BFF, environment separation and deployment runtime | It is the customer/admin frontend and may form a secure thin BFF boundary | Phase 1; auth in 2; gateway in 10; deployment 14 | Build routes with deliberate render/cache choices; keep secrets server-side; later protect sessions and trace a deployed request |
| **Authentication** | Identity proof, sessions, credentials, MFA, lifecycle, authentication vs authorization and account recovery | Protects customer/admin data but does not decide resource ownership | Phase 2; reinforced all APIs | Login/logout/expiry flow; server-side protected access; demonstrate that an authenticated customer still cannot read another order |
| **OAuth 2.0** | Roles, authorization code + PKCE, access/refresh tokens, scopes, audience, state, redirect risks; OAuth is delegated authorization | Auth0 issues API access tokens and browser flow must resist interception/CSRF | Phase 2 | Draw and inspect the authorization flow; validate audience/scope; test altered state, wrong audience and expired token behavior |
| **OpenID Connect** | ID token, `sub`, issuer, nonce, discovery, UserInfo and how OIDC adds authentication to OAuth | Stable Auth0 identity links to User Service; ID tokens must not be mistaken for arbitrary API authorization | Phase 2–3 | Validate login claims/nonce; map `sub` idempotently; show why email is not the durable business identifier |
| **JWT** | Signed vs encrypted, header/payload/signature, algorithms/JWKS, issuer/audience/time claims, rotation, revocation limits | Services will receive bearer tokens; decoding without verification is a serious vulnerability | Phase 2–3 | Verify JWT at API boundary; rotate/test JWKS behavior; reject wrong issuer/audience/algorithm/expiry and avoid logging tokens |
| **Auth0** | Tenants, applications/APIs, callbacks, Universal Login, actions, roles/permissions, environment configuration, quotas/cost/export | Outsources identity while NexaCommerce retains profiles and authorization rules | Phase 2–3; review 14/16 | Configure a non-production tenant, hosted login and permissions; document settings; provision User profile without storing password |
| **REST APIs** | Resource modeling, methods/status/errors, validation, pagination/filter/sort, versioning, idempotency, concurrency, OpenAPI and compatibility | Every initial service exposes clear contracts before events/gateway add complexity | Phase 3 onward | Implement `API.md` subset with OpenAPI; contract-test clients; evolve one field compatibly and handle validation/error codes consistently |
| **Microservices** | Bounded responsibility, data ownership, deployment/autonomy tradeoffs, coupling, discovery, failure, modular monolith alternatives | A project objective, but premature service-per-entity design would overwhelm one learner | ADR/Phase 0; services 3–8 | Implement one owner at a time; prohibit cross-DB access; measure a network failure; justify retaining, splitting, or merging a boundary |
| **PostgreSQL** | Relational design, constraints, transactions/isolation/locks, indexes/query plans, migrations, connections, backups/restores | Durable correctness for users, catalog, orders, stock and payment metadata | Phase 3 onward; deep in 7/14/16 | Own DB/migrations per service; test transaction/concurrency; inspect an explain plan; back up and restore a non-production database |
| **Redis** | In-memory structures, TTL/eviction, cache-aside, invalidation, stampedes, persistence limits, distributed coordination risks | May improve measured catalog latency or counters, but must not become commerce truth | Only when measured, likely after 4/10; operate 14–15 | Baseline first; add one bounded cache/counter; force eviction/outage/stale data; verify fallback and hit/miss/latency metrics |
| **Distributed systems** | Partial failure, latency, timeouts, retries/backoff/jitter, idempotency, clocks, consistency, sagas, compensation and fallacies | Checkout spans independently failing order, inventory, payment and provider boundaries | Introduced 3; concentrated 6–10; hardened 16 | Inject timeout/duplicate/crash; keep checkout in pending/recoverable state; reconcile and correlate the workflow without double charge/order |
| **Message brokers** | Queue vs log, producer/consumer, ack, routing, ordering, retention/replay, backpressure, retries/DLQ and delivery guarantees | Non-blocking notifications and reliable business fact propagation arrive only with a real need | Phase 9 | Compare RabbitMQ/Kafka/managed options; publish through outbox; stop/restart consumer; observe lag, duplicate handling and DLQ replay |
| **Event-driven architecture** | Events as immutable facts, ownership, envelopes/schema evolution, eventual consistency, outbox/inbox, idempotent projections | Decouples post-order work but can obscure truth and leak data if poorly designed | Phase 9; observed 15 | Define versioned `OrderConfirmed`; implement outbox and notification consumer; process duplicate/out-of-order events and evolve schema compatibly |
| **Docker** | Images vs containers, layers, build contexts, multi-stage builds, users/filesystems, networking, volumes, health and signals | Creates reproducible packages after runtime behavior is understood | Phase 11 | Build minimal non-root images; run stack in Compose; inspect networking/volumes; stop gracefully and scan image/secrets |
| **Testing** | Unit/integration/contract/component/E2E/property/load/security/accessibility tests, isolation, fixtures, determinism and test economics | Commerce failures involve money, stock, identity and service contracts; late testing is unsafe | Every phase; strategy expansion 12 | Add risk-focused tests each phase; real-DB integration and contracts; automate critical purchase; seed a defect/failure to prove tests detect it |
| **CI/CD** | Trigger/stages, caching, reproducibility, artifact promotion, secrets/OIDC, approvals, migrations, rollout/rollback and supply chain | Turns local quality into repeatable delivery without granting excessive production power | Phase 13 | Required PR checks; pinned actions; build once/promote artifact; staged deploy with approval and tested rollback using short-lived credentials |
| **Cloud** | Shared responsibility, IAM, networks, managed compute/data, DNS/TLS, KMS/secrets, scaling, availability, backup, cost and IaC | Production orientation requires operating dependencies securely, not just writing services | Evaluate 0/13; implement 14 | Compare providers; provision isolated staging with IaC; deny public DB access; deploy, observe spend, restore backup and safely tear down resources |
| **Observability** | Logs/metrics/traces, correlation/context, RED/USE, SLIs/SLOs/error budgets, alert quality, sampling, retention and telemetry privacy | Cross-service/provider failures cannot be diagnosed from a browser error or raw logs alone | Basic logs in 3; events in 9; full Phase 15 | Trace checkout; dashboard rate/error/duration; trigger a symptom alert; follow runbook; prove tokens/addresses are redacted |
| **Security** | Threat modeling, least privilege, secure defaults, input/output controls, OWASP risks, secrets, dependency/supply chain, encryption, privacy, incident response | Identity, admin, addresses, money and payment integrations make security a continuing design property | Every phase; gates 2/8/10/13/14; hardening 16 | Threat-model auth/checkout; test authorization and webhook replay; rotate a secret; run scans; validate retention/deletion and execute incident exercise |

## 2. Phase learning evidence

At each phase checkpoint, produce only useful evidence:

1. **Concept map:** request/data flow and trust/data owner boundaries.
2. **Decision:** tradeoff made, alternatives, and trigger to reconsider.
3. **Experiment:** a small spike or failure injection when behavior is uncertain.
4. **Tests:** one normal, one invalid/unauthorized, and one relevant failure/concurrency case.
5. **Operations note:** how to run, observe, stop, and recover the increment.
6. **Teach-back:** explain the implementation without reading generated code line by line.

## 3. Suggested reflection questions

- What is authoritative, and what is a cache/snapshot/projection?
- Which trust boundary validates this value or identity?
- What happens if the dependency times out after succeeding?
- Is the retry safe, and how is duplicate work detected?
- Which local transaction protects the invariant? What cannot be atomic?
- What would logs/metrics/traces show without leaking sensitive data?
- What evidence justifies the next service or technology?
- Could a simpler implementation teach the same concept and meet the requirement?

## 4. Completion standard

The curriculum is successful when the developer can design, implement, test, deploy, diagnose, and explain NexaCommerce’s critical journey and its tradeoffs. Completing every technology checkbox is not success; unnecessary technology should remain unimplemented and documented as such.
