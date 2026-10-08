# NexaCommerce Data Architecture

- **Document status:** PLANNED baseline
- **Implementation status:** NOT YET IMPLEMENTED
- **Design maturity:** Preliminary entity outlines; not schemas or migrations
- **Last reviewed:** 2026-10-02

## 1. Principles

1. A service is the authority for its business data and invariants.
2. Other services use its API or published events, never its tables.
3. Transactions are local to one service database; cross-service consistency is explicit and generally eventual.
4. Persist only data needed for product, legal, security, or operational purposes.
5. Schema changes are versioned, reviewed, tested, observable, and reversible/forward-recoverable.
6. PostgreSQL is the durable relational system of record. Redis is not a source of truth.

“Database per service” means logical ownership and isolated credentials. During early local learning, separate databases may share one PostgreSQL server/cluster to control cost. Separate physical clusters are an operational choice, not a prerequisite for ownership.

## 2. Ownership and boundaries

| Owner | Durable data | Does not own |
|---|---|---|
| Auth0 | Credentials, identity-provider records, authentication factors | Commerce profile, addresses, orders |
| User Service | Application user profile, Auth0 subject mapping, addresses, application status | Passwords, order records |
| Product Service | Products, categories, product publication, preliminary reviews | Stock counts, carts, orders |
| Cart Service | Active carts and cart items | Authoritative product price/stock |
| Order Service | Orders, order-item snapshots, checkout workflow/state history | Product master, inventory ledger, payment-provider truth |
| Inventory Service | On-hand/reserved quantities, reservations, adjustment ledger | Product descriptions, orders |
| Payment Service | Payment attempts/status, provider references, processed webhook IDs | Raw card data, order line items |
| Notification capability | Delivery preferences/status where needed | User identity, order truth |

Boundaries may evolve. A capability can begin as a module and later receive an isolated deployment/database, but ownership must remain unambiguous.

## 3. Transactional and cross-service consistency

A local transaction can atomically update only the owning service’s records. Foreign IDs across services are logical references, not cross-database foreign keys. Services may keep explicitly labeled, event-fed read models for display/queries; those copies are not authoritative.

Checkout requires a workflow rather than a distributed database transaction:

1. Order captures validated product/price/address snapshots.
2. Inventory creates an idempotent, expiring reservation.
3. Payment creates an idempotent provider attempt.
4. Success commits the order/reservation; failure releases stock; unknown outcomes remain pending and are reconciled.
5. Retries, duplicate messages/webhooks, and process crashes must be safe.

The exact ordering is decided during Order/Inventory/Payment design after provider semantics are known. A saga/process manager and transactional outbox are recommended when asynchronous coordination is introduced. Two-phase commit is not planned.

## 4. Preliminary entities

Fields are conceptual and may evolve. All durable entities normally have opaque `id`, timestamps, and a concurrency/version strategy. Personal and audit fields require retention/access review.

### User — User Service

- Auth0 subject (unique external identity key), display name, email copy if required, email verification projection if required, locale, application status, created/updated timestamps.
- Auth0 remains authoritative for credentials and core identity claims. Decide how provider profile changes synchronize.
- Do not make email the immutable cross-service identifier.

### Address — User Service

- User ID, recipient name, address lines, locality, administrative area, postal code, country code, phone only if fulfillment requires it, default flags.
- Validate according to initially supported markets; orders store an immutable address snapshot.
- “Delete” may remove it from future selection while retained order snapshots remain under order policy.

### Product — Product Service

- SKU (unique business key), name, slug, description, price amount/currency, publication status, media references, attributes, category associations, timestamps/version.
- Inventory is not stored as an authoritative product field. Publication requires defined completeness rules.

### Category — Product Service

- Name, slug, description, optional parent category ID, display order, status.
- Hierarchy depth/cycle rules are implementation decisions; do not design an unrestricted taxonomy prematurely.

### Review — Product Service initially, post-MVP

- Product ID, author’s logical user ID, order/order-item evidence reference, rating, title/body, moderation/publication status, timestamps/version.
- One eligible review per purchased item/product rule and deletion/moderation policy are TBD.
- Split to a dedicated capability only if moderation, ownership, or scale becomes independently complex.

### Cart — Cart Service

- Owner user ID, status (`active`, `converted`, `expired` as preliminary states), currency, expiry/activity timestamps, version.
- At most one active cart per owner/currency is a possible initial invariant.

### CartItem — Cart Service

- Cart ID, product ID, quantity, optional display snapshot, added/updated timestamps.
- Any price/name snapshot is advisory. Checkout re-queries/revalidates authoritative values.
- Unique cart/product constraint may merge quantity; quantity caps prevent abuse.

### Order — Order Service

- Customer logical ID, human-safe order reference, order/payment/fulfillment statuses, currency, subtotal/tax/shipping/discount/total values, shipping/billing address snapshots, checkout idempotency key/hash, timestamps/version.
- State transitions are explicit. Provider references do not expose secrets.

### OrderItem — Order Service

- Order ID, product ID reference, SKU/name snapshot, unit price/currency, quantity, line totals, tax/discount snapshots where applicable.
- Snapshot fields preserve what was purchased even when the catalog changes.

### Inventory — Inventory Service

- Product ID logical reference, location ID if/when multiple locations exist, on-hand quantity, reserved quantity or derivable reservation balance, version.
- Available is derived (`onHand - reserved`) under defined rules; quantities cannot violate invariants.
- Adjustments use an append-only reasoned ledger even if a balance row accelerates reads.

### Inventory reservation (supporting entity)

- Order/checkout reference, product ID, quantity, status, idempotency key, expiry, created/updated timestamps.
- Expiry/release/commit operations are idempotent and auditable.

### Payment — Payment Service

- Order ID logical reference, provider, opaque provider payment reference, amount/currency, attempt/status, idempotency key, failure category (sanitized), timestamps/version.
- Store no PAN, CVV, raw provider secret, or unnecessary provider payload. Retain only safe metadata needed for reconciliation/support.

### Processed provider event (supporting entity)

- Provider event ID, payload hash/metadata as allowed, processing status/timestamps.
- Enforces webhook deduplication and supports reconciliation without storing prohibited data.

## 5. Relationships across services

- Use immutable opaque IDs in API/event contracts; an owning service validates referenced resources where required.
- No joins, foreign keys, views, triggers, shared ORM models, or shared database credentials across service boundaries.
- Composite screens are assembled through API composition or purpose-built read models. Avoid synchronous call chains for every field.
- Events carry the minimum stable facts consumers need, not entire internal rows. Consumers tolerate duplicate, delayed, and out-of-order delivery.
- Deletion and identifier remapping propagate through explicit policy/events; they are not cascading database deletes.

## 6. Migrations

- Each service owns an ordered migration history and runs migrations using a separately privileged deployment identity.
- Review generated SQL even if an ORM tool creates it. Test against realistic data volume and restore copies.
- Prefer expand/migrate/contract: add compatible structures, deploy compatible code, backfill observably, then remove old structures after all readers move.
- Avoid long blocking table rewrites. Define rollback or forward-fix strategy and backup checkpoints for risky changes.
- Never edit an already-applied production migration; append a corrective migration.
- ORM/tool choice remains PROPOSED in `ARCHITECTURE.md`.

## 7. Backups and recovery

Before production:

- Choose service-specific RPO/RTO from business impact; do not claim values before deployment/provider choices.
- Enable encrypted automated backups and point-in-time recovery where justified, with access separated from normal service credentials.
- Document restore order and external reconciliation for orders/payments/inventory.
- Run and record restore exercises in a non-production environment. A successful backup job alone does not prove recoverability.
- Ensure deletion/retention requirements also apply to backups, logs, event payloads, and cache copies.

## 8. Indexing and query design

- Index primary/unique keys and measured access paths such as Auth0 subject, SKU/slug, order customer plus creation time, provider reference, reservation expiry, and idempotency keys.
- Composite index order follows filter/sort behavior. Partial indexes may support active/published records.
- Every index has write/storage cost. Use query plans and representative data; do not index every column speculatively.
- Paginate collections with deterministic ordering, cap queries, avoid N+1 access, and monitor slow queries/connection saturation.
- PostgreSQL text search is the initial recommendation; adopt a search service only after relevance/scale evidence.

## 9. Redis and cached/ephemeral data

Redis is **PLANNED but not required initially**. Appropriate candidates include short-lived catalog cache entries, rate-limit counters, sessions only if the chosen frontend pattern requires them, and coordination with carefully defined semantics. It must not be authoritative for orders, payments, inventory, or user profiles. Every cache needs ownership, key/version conventions, TTL, invalidation/failure behavior, memory bounds, and sensitive-data review. See ADR-004.

## 10. Retention and privacy

Retention depends on target jurisdiction and business policy and is an open decision. Before real data:

- Classify profile/address, order, payment metadata, audit, telemetry, event, and backup data.
- Define purpose, legal basis where applicable, access, retention, deletion/anonymization, and legal-hold behavior by class.
- Minimize replicated personal data and event payloads; redact logs and traces.
- Preserve financial/order facts only as required while separating or anonymizing customer-identifying fields when policy permits.
- Test subject-access/deletion workflows across service boundaries before public launch.

## 11. Decisions deferred to implementation

Database naming/topology, ORM/query tool, ID format, exact money representation, isolation/locking details, migration runner, RPO/RTO, retention durations, inventory locations, tax model, and provider-specific payment fields remain **PROPOSED/TBD**. They require phase-specific ADRs or contract updates; this document is not permission to generate schemas ahead of those decisions.
