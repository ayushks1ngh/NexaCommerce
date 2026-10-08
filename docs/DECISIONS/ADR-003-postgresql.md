# ADR-003: PostgreSQL as the primary durable datastore

- **Status:** ACCEPTED; databases and schemas are NOT YET IMPLEMENTED
- **Date:** 2026-10-02

## Context

Commerce data includes products, customers, carts, orders, stock, and payment metadata with constraints, auditable state transitions, and transactional updates. The project must teach relational modeling, transactions, migrations, indexing, backup, and operational discipline. Services own their data.

## Problem

Which datastore should be the default durable system of record while preserving service ownership and allowing later specialization where evidence supports it?

## Options considered

1. **PostgreSQL:** mature relational database with ACID transactions, constraints, indexing, JSON support, and broad managed hosting.
2. **MySQL/MariaDB:** capable relational alternatives with broad tooling.
3. **Document database:** flexible document model and horizontal scaling options.
4. **Distributed SQL:** relational semantics with built-in distribution at greater cost/complexity.
5. **Polyglot persistence from the start:** select a different store for each service.

## Decision

Use **PostgreSQL as the default durable relational datastore** for initial services. Each service owns a logical database (or equivalently isolated database/schema only if credentials and access prevent cross-service use) and its migrations. Early local environments may share one PostgreSQL server; no service may query another service’s data directly.

PostgreSQL is a default, not a permanent mandate for every future workload. Any specialized datastore requires measured need and an ADR. ORM/query tool and physical hosting remain undecided.

## Rationale

- Transactions and constraints fit orders, payments, reservations, and inventory invariants.
- Relational queries suit early catalog and administrative workflows.
- Mature ecosystem, operational knowledge, portability, and managed-cloud availability.
- Native indexing, JSONB, and text-search features allow the MVP to avoid premature search/document infrastructure.
- One database technology lowers learning and operational load while service boundaries are still evolving.

## Consequences

### Positive

- Strong local consistency and explicit schemas.
- Common migration, backup, observability, and troubleshooting skills transfer across services.
- Can support the expected initial workload without speculative infrastructure.

### Negative and risks

- Schema/migration discipline is required; careless changes can block or corrupt workloads.
- Connection counts multiply with services and require limits/pooling.
- Database-per-service increases migration, backup, and restore responsibilities.
- PostgreSQL does not eliminate distributed consistency problems across services.
- Some future search, analytics, time-series, or globally distributed workloads may need specialized systems.

### Required mitigations

- Least-privilege service credentials and no cross-service joins/foreign keys.
- Versioned migrations with expand/contract practices, representative testing, and restore drills.
- Index from observed query patterns; monitor slow queries and connection saturation.
- Keep event/API contracts separate from table schemas.

## Alternatives rejected

- **MySQL/MariaDB:** viable, but offers no compelling initial advantage over PostgreSQL for this project’s goals.
- **Document database as default:** flexible schema is not worth weakening or reimplementing relationships and transactional invariants for this domain.
- **Distributed SQL:** rejected until geographic scale/availability requirements justify complexity and cost.
- **Polyglot persistence immediately:** rejected as unnecessary cognitive and operational overhead.

## Revisit when

Reconsider per workload if measured search relevance, analytical isolation, write distribution, geographic latency, retention, or data model requirements exceed PostgreSQL’s appropriate use. Scaling should begin with query/schema tuning and managed operational options, not automatic replacement.
