# NexaCommerce Product Requirements Document

- **Document status:** DECIDED as the planning baseline; requirements remain subject to validated learning
- **Product status:** NOT YET IMPLEMENTED
- **Last reviewed:** 2026-10-02
- **Owner:** Product and engineering

## 1. Product overview

NexaCommerce is a learning-focused, production-oriented commerce platform through which customers discover products, purchase them, and manage orders while administrators maintain the catalog, inventory, customers, and fulfillment state. It will be built incrementally so each capability teaches the engineering concept that supports it. “Production-oriented” describes design intent and engineering discipline, not a claim that the system is production-ready.

## 2. Problem statement

Customers need a coherent and trustworthy path from product discovery to order tracking. Operators need accurate catalog, stock, and order controls. The developer needs to learn modern web and distributed-system engineering without hiding fundamentals behind a generated, over-engineered platform. The product must therefore deliver useful commerce flows while exposing clear boundaries, tradeoffs, tests, and operational concerns at a manageable pace.

## 3. Vision

Build the smallest understandable commerce platform that can evolve safely from a local application into an observable, secure cloud system, with every architectural addition justified by a product or operational need.

## 4. Goals

1. Deliver an end-to-end customer journey: discover, cart, checkout, pay in a sandbox, and track an order.
2. Give an authorized administrator the minimum controls needed to operate that journey.
3. Keep identity, business data, and service ownership explicit.
4. Teach technologies through working increments that are designed, tested, and documented.
5. Establish measurable quality targets before public production use.

## 5. Non-goals

- Marketplace/multiple sellers, auctions, subscriptions, social commerce, recommendations, loyalty, gift cards, or native mobile apps in the MVP.
- Multi-region active-active operation, unlimited scale, or regulatory certification during initial learning phases.
- Building identity, card processing, search, email, or cloud infrastructure from first principles when a safe external service is appropriate.
- Creating one service per entity or introducing a broker, gateway, cache, containers, or Kubernetes before its need is demonstrated.
- Supporting every country’s tax, shipping, currency, privacy, and accessibility regime initially.

## 6. Target users and personas

| Persona | Need | Product response |
|---|---|---|
| Guest shopper | Evaluate products with little friction | Public catalog, search, filters, details; sign-in required only when necessary |
| Registered customer | Purchase safely and revisit orders | Saved profile/addresses, durable cart, checkout, order history and tracking |
| Catalog administrator | Keep sale information correct | Product/category management with validation and auditability |
| Operations administrator | Prevent overselling and fulfill orders | Inventory and order controls, customer lookup, basic operational status |
| Developer/operator | Learn and operate the platform | Clear docs, tests, logs, metrics, traces, runbooks introduced progressively |

Initial admin personas may share one `admin` role. More granular roles are post-MVP if operational need justifies them.

## 7. Journeys

### 7.1 Customer journeys

**Discover and evaluate:** A guest opens the storefront, browses or searches the catalog, filters results, opens a product, and sees current price, description, imagery, and availability indication.

**Register and manage identity:** The customer signs up or logs in through Auth0, returns to the intended page, and maintains application profile and addresses in NexaCommerce. Credentials remain in Auth0.

**Purchase:** The customer adds purchasable quantities to a cart, reviews totals, selects an address, enters payment through a hosted/tokenized sandbox payment flow, confirms the order, and receives a durable order identifier. The flow must handle stock, price, and payment changes without silent inconsistency.

**After purchase:** The customer sees order history and status, opens tracking details when available, and can later leave a review for an eligible purchased product.

### 7.2 Admin journeys

**Manage catalog:** An authorized admin creates or edits product information, assigns categories, changes publication status, and can diagnose validation failures.

**Manage stock:** An authorized admin views available/reserved quantities and records justified adjustments without directly editing another service’s database.

**Manage orders:** An admin finds an order, reviews its timeline, and applies permitted status transitions for fulfillment or support.

**Support a user:** An admin searches application profiles and order references, suspends application access where policy permits, but cannot retrieve passwords or payment credentials.

**Observe operations:** An admin/operator views basic health and actionable product signals such as failed payments or low stock; full analytics is not an MVP requirement.

## 8. Core features and reasons

| Capability | Scope | Reason |
|---|---|---|
| Registration/login | MVP | Establishes customer identity and protects personal/order data |
| Product browsing, search, filtering, details | MVP | Enables product discovery; search may initially use PostgreSQL |
| Cart | MVP | Allows intent to persist before purchase |
| Checkout and sandbox payment | MVP | Completes the core business journey without handling raw card data |
| Addresses | MVP | Required for a realistic checkout; locale rules remain constrained |
| Order history and status tracking | MVP | Gives customers confidence after payment |
| Product management | MVP | Makes the catalog operable without database edits |
| Inventory management | MVP | Supports availability and oversell prevention |
| Order management | MVP | Supports basic fulfillment operations |
| User management | MVP, minimal | Enables support and application-level suspension/role visibility |
| Basic platform visibility | MVP, minimal | Enables operators to detect failure; not a business-intelligence suite |
| Reviews | Post-MVP | Builds trust, but is not required to prove the transaction flow |

## 9. Functional requirements

Status for all requirements below is **PLANNED** unless stated otherwise.

### 9.1 Customer

- **FR-C01 Identity:** A user can register, log in, log out, and recover access through Auth0.
- **FR-C02 Catalog:** A user can browse published products and categories.
- **FR-C03 Search:** A user can search published products using a documented initial matching strategy.
- **FR-C04 Filter/sort:** A user can filter by supported category, price, and availability fields and sort by supported options.
- **FR-C05 Details:** A user can view product description, media references, current price, and availability indication.
- **FR-C06 Cart:** A user can add, update, remove, and view cart items. Invalid or unavailable quantities produce actionable feedback.
- **FR-C07 Cart reconciliation:** Checkout revalidates price, publication status, and stock rather than trusting stale cart data.
- **FR-C08 Address:** An authenticated customer can create, update, list, select, and delete their own addresses subject to order-retention rules.
- **FR-C09 Checkout:** An authenticated customer can submit a checkout request idempotently and receive a clear success, pending, or failure outcome.
- **FR-C10 Payment:** Payment is handled by a compliant external provider using tokens/hosted fields; NexaCommerce stores no raw card details.
- **FR-C11 Orders:** A customer can view only their own order history and order details.
- **FR-C12 Tracking:** A customer can view the current order status and, when integrated, carrier tracking reference.
- **FR-C13 Reviews:** Post-MVP, an eligible customer can create/update one review per purchased product; moderation rules must be defined before release.

### 9.2 Administration

- **FR-A01 Authorization:** Admin functions require authenticated users with an authorized application role/permission.
- **FR-A02 Products:** Admins can create, update, publish/unpublish, and list products and manage category assignment.
- **FR-A03 Inventory:** Admins can view stock and create attributable stock adjustments; reservation changes occur through inventory workflows.
- **FR-A04 Orders:** Admins can search orders and apply only valid, attributable status transitions.
- **FR-A05 Users:** Admins can find application profiles, inspect application status/roles, and suspend/restore application access according to policy.
- **FR-A06 Visibility:** Admins can see basic service health and operational exceptions without access to secrets or unnecessary personal data.

### 9.3 Cross-cutting business rules

- **FR-X01 Money:** Monetary values use an explicit currency and non-floating representation; initial supported currency is TBD.
- **FR-X02 Snapshot:** Orders retain product name, unit price, currency, quantity, address, and total snapshots needed for historical accuracy.
- **FR-X03 Inventory:** Inventory changes are auditable; oversell behavior and reservation expiry are defined before real payment use.
- **FR-X04 Idempotency:** Checkout, payment creation, provider webhooks, and other retry-prone writes are idempotent.
- **FR-X05 Statuses:** Order and payment state transitions are explicit and validated.
- **FR-X06 External failures:** Provider timeouts and delayed callbacks produce recoverable/pending states, not fabricated success.

## 10. Non-functional requirements

These are design requirements. Numeric values are **PROPOSED acceptance targets**, not measured claims, and must be baselined under a documented workload before launch.

### 10.1 Security

- **NFR-S01:** Enforce TLS outside local development; encrypt managed data at rest where the platform supports it.
- **NFR-S02:** Validate JWT issuer, audience, signature, expiry, and permissions server-side; never trust UI role checks alone.
- **NFR-S03:** Apply least privilege to users, services, databases, deployment identities, and admin operations.
- **NFR-S04:** Validate and constrain all inputs; encode outputs; use parameterized persistence APIs and safe file/media handling.
- **NFR-S05:** Keep credentials and secrets out of source control, logs, browser bundles, and images; establish rotation procedures.
- **NFR-S06:** Do not store raw passwords or card data. Verify signed payment webhooks and prevent replay.
- **NFR-S07:** Protect browser flows against relevant CSRF, XSS, clickjacking, session, and redirect attacks; define CORS narrowly.
- **NFR-S08:** Avoid logging tokens, secrets, full addresses, or payment data; redact sensitive fields and audit privileged changes.
- **NFR-S09:** Add dependency, secret, and source scanning in CI before production deployment.
- **NFR-S10:** Complete threat modeling and privacy/retention review before handling real customer/payment data.

### 10.2 Performance

- **NFR-P01:** Proposed initial target: p95 server API latency below 500 ms for ordinary read endpoints under the agreed baseline, excluding external-provider time.
- **NFR-P02:** Proposed initial target: p95 catalog page server response below 1 second under the agreed baseline; user-centric Core Web Vitals will be measured separately.
- **NFR-P03:** Paginate unbounded collections and cap page size; avoid unbounded queries and N+1 access.
- **NFR-P04:** Set explicit timeouts for network calls. Add caching only after measuring latency/load and defining invalidation.
- **NFR-P05:** Establish workload, dataset size, environment, and percentile definitions before treating targets as release gates.

### 10.3 Reliability

- **NFR-R01:** Proposed initial availability objective is 99.5% monthly for the customer purchase path after production launch; maintenance and measurement rules remain TBD.
- **NFR-R02:** Durable writes use transactions within one service; cross-service workflows explicitly handle partial failure and retries.
- **NFR-R03:** Retry only transient failures with bounded backoff/jitter; asynchronous consumers and webhooks are idempotent.
- **NFR-R04:** Define backup, restore, recovery point, and recovery time objectives before production; test restoration rather than assuming backups work.
- **NFR-R05:** Health/readiness signals, structured logs, metrics, traces, alerts, and runbooks are introduced before production readiness.
- **NFR-R06:** Deployments must support rollback or roll-forward and preserve backward compatibility during staged changes.

### 10.4 Accessibility

- **NFR-A11Y01:** Target WCAG 2.2 AA for customer and admin web experiences.
- **NFR-A11Y02:** Use semantic structure, keyboard-operable controls, visible focus, adequate contrast, text alternatives, and programmatic labels.
- **NFR-A11Y03:** Validation and status changes are understandable without color alone and announced to assistive technology when appropriate.
- **NFR-A11Y04:** Checkout supports zoom/reflow and avoids unnecessary time limits; payment-provider accessibility is evaluated during selection.
- **NFR-A11Y05:** Automated checks supplement, but do not replace, keyboard and screen-reader testing of critical journeys.

### 10.5 Maintainability and operability

- Public contracts and architectural decisions are versioned with code; services expose ownership and run instructions.
- Type checks, linting, focused unit/integration tests, and critical journey tests become automated gates progressively.
- Logs use correlation identifiers and consistent error codes; telemetry must not expose sensitive data.

## 11. Scope

### 11.1 MVP — PLANNED

The MVP is the first coherent sandbox transaction and operator loop:

- Responsive storefront; catalog browsing, basic search/filter/sort, and product details.
- Auth0 registration/login/logout and protected customer/admin routes.
- Customer profile and addresses.
- Persistent customer cart with checkout-time reconciliation.
- Order creation, inventory validation/reservation, sandbox payment, confirmation, history, and status.
- Minimal admin product/category, stock adjustment, order transition, and user lookup/suspension controls.
- Baseline security controls, tests for critical behavior, CI, documented cloud deployment, backup/restore exercise, and actionable telemetry.
- One locale, one TBD currency, one simple shipping/tax policy defined before checkout implementation.

### 11.2 Post-MVP — PLANNED AFTER MVP EVIDENCE

- Verified-purchase reviews and moderation.
- Refund/cancellation workflows and richer fulfillment/carrier tracking.
- Notification preferences and production email integration.
- Better catalog search only if PostgreSQL search no longer meets measured needs.
- More granular admin roles/audit views and product media management.
- Guest-cart merge and guest checkout if user research supports them.

### 11.3 Future — PROPOSED, NOT COMMITTED

- Multiple currencies/locales, jurisdiction-aware tax and shipping providers.
- Promotions, wish lists, recommendations, analytics, marketplace sellers, native applications, and multi-region resilience.
- Each requires a validated user/business need and separate decision; none should shape the initial architecture prematurely.

## 12. Success metrics

No baseline exists. During implementation, instrument and baseline the following; targets require product validation rather than invention:

| Outcome | Metric | Initial use |
|---|---|---|
| Purchase journey works | Checkout completion rate and failure reasons in sandbox/usability sessions | Detect product and technical blockers |
| Discovery is useful | Search zero-result rate and product-detail progression | Improve catalog findability |
| Orders are trustworthy | Paid orders with consistent payment/inventory/order state; reconciliation exceptions | Target zero unexplained inconsistencies |
| Platform is supportable | Time to detect/recover from seeded failures; restore exercise result | Establish operational readiness |
| Experience is usable | Critical-journey accessibility findings and task completion | Gate known critical barriers |
| Learning succeeds | Each roadmap phase has notes, tests, ADR updates, and explainable tradeoffs | Verify understanding, not code volume |

## 13. Assumptions

- One developer is the initial builder/operator and uses sandbox/non-sensitive data until security and operations gates pass.
- Auth0 is available and acceptable for identity; business profiles remain in the User Service.
- PostgreSQL is the system of record for relational service data. Redis is optional until a concrete cache/ephemeral-state need exists.
- A third-party payment provider will isolate card handling; provider choice is open.
- Initial catalog and traffic fit straightforward PostgreSQL-backed search and a single-region deployment.
- Reviews are required product capability but are post-MVP so the transactional core is learned first.

## 14. Constraints

- Technology breadth must not replace understanding; phases are implemented independently and reviewed before advancing.
- Service ownership prohibits cross-service database reads/writes.
- Budget, cloud/provider accounts, target market, data jurisdiction, currency, tax, shipping, refund policy, and launch traffic are unknown.
- This Phase 0 produces documentation only. All features and infrastructure remain NOT YET IMPLEMENTED.
- External-provider limits, terms, availability, accessibility, and data residency will constrain later choices.

## 15. Open product decisions

Before checkout implementation, decide target market, currency, tax display/calculation, shipping options, address validation, cancellation/refund policy, inventory reservation timeout, and payment sandbox provider. Before public launch, decide privacy jurisdiction, retention schedule, support process, real SLOs/error budgets, accessibility test scope, and incident ownership.
