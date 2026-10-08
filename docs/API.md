# NexaCommerce API Contract Baseline

- **Document status:** PLANNED
- **Implementation status:** NOT YET IMPLEMENTED
- **Scope:** Contract conventions and restrained endpoint outline; not a complete OpenAPI specification
- **Last reviewed:** 2026-10-02

## 1. Principles

1. Model business resources and workflows, not database tables.
2. Use JSON over HTTPS and clear REST semantics initially.
3. Keep contracts consistent while services retain ownership and autonomy.
4. Validate at trust boundaries and return machine-readable, safe errors.
5. Make retry-prone operations idempotent.
6. Paginate collections and constrain expensive queries.
7. Evolve compatibly; document and test contracts before consumers depend on them.
8. Do not expose internal service endpoints directly to public clients in the target architecture.

Initial development may call one service directly while learning. A gateway/BFF is introduced only when multiple public APIs create a concrete routing/security/aggregation need.

## 2. Conventions

### 2.1 URLs

- External base: `/api/v1` once a gateway/BFF exists; service-local development may use `/v1`.
- Lowercase plural nouns: `/products`, `/orders/{orderId}`.
- Use opaque stable IDs, not mutable names, in paths.
- Nest only when ownership/context is clear: `/users/me/addresses`.
- Workflows that are not natural CRUD use explicit action resources, for example `/orders/{id}/cancellations` rather than RPC-like verbs.
- Query parameter names use `camelCase`; JSON properties use `camelCase`; timestamps use UTC RFC 3339.
- Money is represented as `{ "amount": "19.99", "currency": "USD" }` until a shared contract chooses minor-unit representation. Floating-point values are forbidden.

### 2.2 Methods

| Method | Use | Idempotent expectation |
|---|---|---|
| `GET` | Read resource/collection | Yes |
| `POST` | Create resource or submit command | No by default; require idempotency for checkout/payment operations |
| `PUT` | Full replacement where supported | Yes |
| `PATCH` | Partial constrained update | Idempotent for the same document where practical |
| `DELETE` | Delete/deactivate where policy permits | Yes |

### 2.3 Success status codes

- `200 OK`: successful read/update/action with body.
- `201 Created`: resource created; include `Location` when externally addressable.
- `202 Accepted`: asynchronous work accepted but not complete.
- `204 No Content`: successful operation with no response body.

### 2.4 Error status codes

- `400` malformed syntax/parameters; `401` missing or invalid authentication; `403` authenticated but not authorized; `404` resource absent or deliberately concealed; `409` state/version/idempotency conflict; `412` failed precondition; `422` semantically invalid input; `429` rate limited; `500` unexpected failure; `502/503/504` dependency/unavailable/timeout conditions.
- Do not use `200` for errors or reveal stack traces, secrets, internal hostnames, or sensitive records.

### 2.5 Error format

Use `application/problem+json` aligned with RFC 9457:

```json
{
  "type": "https://docs.nexacommerce.example/problems/validation-error",
  "title": "Request validation failed",
  "status": 422,
  "code": "VALIDATION_ERROR",
  "detail": "One or more fields are invalid.",
  "instance": "/api/v1/products/request-reference",
  "traceId": "opaque-correlation-id",
  "errors": [
    { "field": "price.amount", "code": "INVALID_MONEY", "message": "Enter a valid non-negative amount." }
  ]
}
```

`code` is stable for consumers; `detail` and field messages are human-readable and safe. `traceId` supports diagnostics but reveals no topology. The public problem type domain is a placeholder until a real documentation domain exists.

## 3. Validation and concurrency

- Reject unknown or immutable fields where accepting them could hide client errors or mass-assignment risks.
- Define length, format, range, enum, and collection limits in a future OpenAPI contract; normalize only when semantics are unambiguous.
- Validate business rules in the owning service even if the UI already validates them.
- Use ETags/version fields with `If-Match` for admin updates where lost updates matter; return `409` or `412` consistently.
- `POST /checkouts` and payment-affecting writes require an `Idempotency-Key`; same key plus different payload is a conflict. Retention duration is decided with implementation.

## 4. Authentication and authorization

- Public catalog reads may be anonymous. Customer, checkout, and admin APIs require `Authorization: Bearer <access-token>` issued by Auth0 for the expected API audience.
- Every receiving service validates signature through cached JWKS, issuer, audience, expiry/not-before, algorithm, and required scopes/permissions. Authentication is never inferred from user-supplied IDs.
- Prefer `/users/me` and derive the subject from the token. For resources such as orders, enforce ownership server-side.
- Admin endpoints require explicit permissions such as `products:write`, not merely a hidden admin page.
- Service-to-service credentials and authorization are separate from customer tokens; the mechanism is a later security decision.
- Tokens and personal/payment data must not be logged. Rate limits apply by route and principal at an eventual gateway, with defense in depth for sensitive services.

## 5. Collections

### Pagination

Cursor pagination is the recommended default for changing/high-volume collections:

`GET /products?limit=20&cursor=<opaque>`

```json
{ "items": [], "page": { "nextCursor": null, "hasMore": false } }
```

`limit` has documented defaults and caps. Offset pagination may be used for small admin lists when stable ordering and cost are acceptable.

### Filtering, searching, and sorting

- Explicit allowlists only: `categoryId`, `minPrice`, `maxPrice`, `availability`, `status`.
- Search uses `q=<term>` with documented matching behavior; it is not promised as a search-engine query language.
- Sort uses `sort=price` or `sort=-createdAt`; stable tie-break by ID.
- Invalid/unsupported filters and sort fields return `400`, rather than being silently ignored.

## 6. Versioning

- Start with path major version `/api/v1` at the public boundary.
- Additive optional fields are normally backward compatible. Never repurpose fields or enum meanings.
- Breaking changes require a new major version, migration/deprecation window, usage evidence, and an ADR.
- Internal event schemas have independent explicit versions; API versioning does not version database schemas.
- A machine-readable OpenAPI description and consumer contract checks are deliverables of service implementation phases, not Phase 0.

## 7. Initial endpoint outline

All entries are **PLANNED** and **NOT YET IMPLEMENTED**. Names and ownership are preliminary; implementation phases must narrow and specify request/response schemas.

### User Service

| Method/path | Access | Purpose |
|---|---|---|
| `GET /v1/users/me` | Customer | Read application profile linked to Auth0 subject |
| `PATCH /v1/users/me` | Customer | Update permitted profile fields |
| `GET/POST /v1/users/me/addresses` | Customer | List/create own addresses |
| `GET/PATCH/DELETE /v1/users/me/addresses/{addressId}` | Customer | Manage own address |
| `GET /v1/admin/users` | `users:read` | Search application profiles |
| `PATCH /v1/admin/users/{userId}/status` | `users:write` | Suspend/restore application access; not Auth0 credentials |

### Product Service

| Method/path | Access | Purpose |
|---|---|---|
| `GET /v1/products` | Public | Browse/search/filter published products |
| `GET /v1/products/{productId}` | Public | View a published product |
| `GET /v1/categories` | Public | List browsable categories |
| `POST /v1/admin/products` | `products:write` | Create product draft |
| `GET/PATCH /v1/admin/products/{productId}` | Admin product permission | Read/update product including unpublished state |
| `POST /v1/admin/products/{productId}/publication` | `products:publish` | Apply an explicit publish/unpublish transition |
| `POST/PATCH /v1/admin/categories...` | `products:write` | Minimal category management; exact routes TBD |

Review endpoints are post-MVP and may remain in Product Service initially: `GET /v1/products/{id}/reviews`, `POST /v1/products/{id}/reviews`, and permitted update/moderation routes. Split only if responsibility/scale warrants it.

### Cart Service

| Method/path | Access | Purpose |
|---|---|---|
| `GET /v1/carts/current` | Customer | Get active cart |
| `POST /v1/carts/current/items` | Customer | Add item |
| `PATCH/DELETE /v1/carts/current/items/{itemId}` | Customer | Change quantity/remove item |
| `DELETE /v1/carts/current` | Customer | Clear active cart |

Cart responses show estimated values. Checkout is authoritative and revalidates product/stock.

### Order Service / checkout orchestration

| Method/path | Access | Purpose |
|---|---|---|
| `POST /v1/checkouts` | Customer + idempotency key | Validate and initiate purchase from current cart |
| `GET /v1/checkouts/{checkoutId}` | Owner | Observe pending/failed/completed result |
| `GET /v1/orders` | Customer | Paginated own order history |
| `GET /v1/orders/{orderId}` | Owner | Own order detail/status timeline |
| `GET /v1/admin/orders` | `orders:read` | Search orders |
| `POST /v1/admin/orders/{orderId}/transitions` | `orders:write` | Apply allowed fulfillment transition |

Cancellation/refund endpoints are post-MVP and require policy/state-machine design.

### Inventory Service

| Method/path | Access | Purpose |
|---|---|---|
| `GET /v1/inventory/{productId}` | Internal | Query availability when synchronous validation is needed |
| `POST /v1/inventory/reservations` | Internal + idempotency | Reserve stock for checkout |
| `POST /v1/inventory/reservations/{id}/commit` | Internal | Convert reservation after successful payment/order transition |
| `DELETE /v1/inventory/reservations/{id}` | Internal | Release reservation idempotently |
| `GET /v1/admin/inventory` | `inventory:read` | View stock |
| `POST /v1/admin/inventory/{productId}/adjustments` | `inventory:write` | Record reasoned stock adjustment |

Reservation API details are deliberately deferred until Order and Inventory phases model failure and expiry.

### Payment Service

| Method/path | Access | Purpose |
|---|---|---|
| `POST /v1/payments` | Internal + idempotency | Create payment attempt for an order |
| `GET /v1/payments/{paymentId}` | Internal/authorized support | Read payment status, never card data |
| `POST /v1/payment-webhooks/{provider}` | Signed provider call | Receive and idempotently process provider event |

Public payment UI integration depends on provider. Refunds are post-MVP.

### Notification capability

Initially this may be an internal module or event consumer, not necessarily a deployed service. If separated, it consumes versioned events and exposes only admin/support delivery-status APIs if a use case arises; no speculative public endpoint is defined.

## 8. Contract governance

Each implemented endpoint must add: owner, threat considerations, request/response schema, examples, authorization rule, idempotency/concurrency behavior, error codes, rate/size limits, tests, and deprecation policy. Contract changes and event schemas are reviewed with consumers; database models never become the public contract by accident.
