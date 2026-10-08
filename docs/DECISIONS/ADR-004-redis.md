# ADR-004: Redis for bounded ephemeral and caching use cases

- **Status:** ACCEPTED WITH CONDITIONS as an allowed supporting technology; adoption is PLANNED only when a use case is demonstrated; NOT YET IMPLEMENTED
- **Date:** 2026-10-02

## Context

NexaCommerce may need low-latency caching, rate-limit counters, short-lived coordination, or other ephemeral data as traffic and distributed behavior develop. Redis is named in the project’s learning goals, but there is no measured load or current feature requiring it. Durable commerce correctness must not depend on cache availability.

## Problem

Should Redis be introduced, when, and for which responsibilities without creating an additional source of truth or premature operational burden?

## Options considered

1. **No cache initially; optimize PostgreSQL/application behavior first.**
2. **Redis introduced for explicit bounded use cases** with TTL, invalidation, and failure semantics.
3. **Application/in-process cache** for single-instance, non-coherent data.
4. **PostgreSQL-backed counters/leases/state** where durability and transactionality matter more than latency.
5. **Redis as a primary datastore or general-purpose message broker.**

## Decision

Do **not deploy Redis at project start**. Adopt it later only for an explicit, measured need and treat it as disposable supporting infrastructure. Initial candidate uses are catalog response/object caching, rate-limit counters, and carefully designed short-lived coordination. Every use must specify owner, key/version format, TTL, invalidation, capacity/eviction policy, consistency impact, sensitive-data classification, outage behavior, and observability.

Redis will not be the authoritative store for products, carts unless separately justified and durability semantics are proven, orders, inventory, payments, identity, or events. PostgreSQL remains the durable record. Redis Streams/pub-sub is not the default event bus decision.

## Rationale

- Preserves the opportunity to learn caching and ephemeral distributed state where it creates visible value.
- Avoids cache invalidation and extra operations before there is evidence of benefit.
- Keeps critical correctness in durable service-owned databases.
- Allows a managed Redis-compatible offering later without choosing a provider now.

## Consequences

### Positive

- Can reduce repeat-read latency/load and support efficient short-lived counters.
- Clear failure boundary: cache loss should degrade performance, not corrupt commerce data.
- Defers cost and operational complexity.

### Negative and risks

- Cache staleness, stampedes, hot keys, eviction, memory pressure, and failover can harm behavior.
- Distributed locks can create false safety if leases, fencing, and failure are misunderstood.
- A new network dependency requires security, monitoring, backup decisions (if any), upgrades, and capacity management.
- Sensitive cached data expands privacy/deletion scope.

### Required mitigations

- Cache-aside or another documented pattern with bounded TTL and versioned keys.
- Graceful fallback, timeouts, jitter, stampede controls, hit/miss/latency/eviction metrics, and memory limits.
- TLS/auth/private networking and least privilege in non-local environments.
- Never use a Redis lock as a substitute for database invariants without a reviewed correctness design.

## Alternatives rejected

- **Redis from day one:** rejected because no measured requirement exists.
- **Redis as primary commerce database:** rejected because eviction/durability/transaction semantics do not fit the initial system-of-record needs.
- **Redis pub/sub as the durable event backbone:** rejected because delivery, replay, and consumer guarantees must be selected during the event phase.
- **Never using Redis:** rejected because bounded cache/counter uses may provide learning and operational value after measurement.

## Revisit when

Adopt/reassess after profiling identifies a latency/load problem or a concrete ephemeral-state requirement. Reconsider the chosen use if hit rate is poor, invalidation causes correctness issues, operations exceed benefit, or a CDN, in-process cache, gateway, PostgreSQL, or broker is a better fit.
