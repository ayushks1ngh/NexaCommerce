# ADR-001: Evolutionary microservices architecture

- **Status:** ACCEPTED as a target direction; initial service extraction is PLANNED and NOT YET IMPLEMENTED
- **Date:** 2026-10-02

## Context

NexaCommerce must teach service boundaries, independent data ownership, REST, events, deployment, and distributed-system operations. Its domains—catalog, cart, ordering, inventory, and payment—have distinct responsibilities and failure modes. However, the project begins with one developer, no users, no measured scale, and no operational platform.

## Problem

How should the system demonstrate realistic microservice practices without multiplying deployments, network failure modes, and duplicated scaffolding before business boundaries are understood?

## Options considered

1. **Single monolith indefinitely:** simplest operation and transactions, but does not meet the project’s explicit distributed-systems learning goal.
2. **Modular monolith first:** enforce modules and ownership in one deployment/database topology, then extract based on learning or operational need.
3. **Immediate fine-grained microservices:** create every potential service from the start.
4. **Evolutionary coarse-grained services:** establish contracts and ownership, implement one capability at a time, and allow modules/services to be extracted incrementally.

## Decision

Adopt **evolutionary, coarse-grained microservices as the target architecture**. Start with the Next.js frontend and add one independently owned backend capability per roadmap phase. A capability may temporarily be an in-process module or share a PostgreSQL server, provided ownership, credentials/databases, contracts, and migration paths remain explicit. Do not create a service merely because an entity exists.

Initial candidate boundaries are User, Product, Cart, Order, Inventory, and Payment. Notification begins as a capability/event consumer and becomes a service only if independent lifecycle, reliability, or scale justifies it. Reviews remain with Product initially. The Order capability may coordinate checkout until orchestration complexity warrants a distinct component.

## Rationale

- Satisfies the learning goal through genuine boundaries rather than a simulated final diagram.
- Aligns boundaries with business responsibility and data authority.
- Keeps each phase runnable and understandable.
- Defers broker, gateway, service mesh, orchestration platform, and independent clusters until concrete needs appear.
- Preserves a path to independent deployment and scaling where failure isolation or team ownership later benefits.

## Consequences

### Positive

- Clear ownership and smaller domain models.
- Practical experience with contracts, eventual consistency, idempotency, observability, and deployment.
- Capabilities can evolve at different rates.

### Negative and risks

- More repositories/packages, processes, network calls, migrations, telemetry, and security boundaries than a monolith.
- Cross-service workflows cannot rely on one ACID transaction.
- Local development and debugging become harder; operational cost rises with each deployment.
- With one developer, productivity may be lower and boundaries may prove wrong.

### Required mitigations

- REST first; asynchronous coordination only for a demonstrated need.
- No cross-service database access or shared domain models.
- Contract tests, correlation IDs, timeouts, idempotency, and documented ownership as services appear.
- Revisit boundaries; merging a poorly justified service is acceptable.

## Alternatives rejected

- **Monolith indefinitely:** rejected because it fails a core educational objective, though modular-monolith techniques remain useful early.
- **Immediate service per listed domain/entity:** rejected as premature infrastructure and operational complexity.
- **Serverless function per endpoint:** rejected as a boundary/deployment model chosen before workload and provider constraints are known.

## Revisit when

Reconsider if maintenance overhead prevents delivery, boundaries cause pervasive synchronous coupling, deployment cost exceeds learning value, or measured organizational/scaling needs suggest different decomposition. The target is not a mandate that every listed service must exist.
