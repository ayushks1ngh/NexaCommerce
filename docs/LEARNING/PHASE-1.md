# Phase 1 Learning Plan — Next.js Frontend Foundation

- **Phase status:** PLANNED / NOT YET IMPLEMENTED
- **Implementation status:** Nothing in this phase has been built. This is a learning **plan**.
- **Last reviewed:** 2026-10-08
- **Related curriculum:** `docs/LEARNING.md`; roadmap entry: `docs/ROADMAP.md` Phase 1

> This document is a **plan**, not a completion report. No code, dependencies, or configuration
> described here exist yet. Each sub-phase will only be implemented after the teach-before-code
> explanation and explicit human approval, following `docs/KIRO_WORKFLOW.md`.

---

## 1. Phase objective (from `docs/ROADMAP.md` Phase 1)

Build a small, accessible web shell and a public catalog experience **using fixture data only** —
with **no backend and no authentication**. The deliverable is a locally runnable, responsive
storefront shell that demonstrates React/Next.js fundamentals, TypeScript boundaries, deliberate
rendering choices, an accessibility baseline, and visible empty/loading/error states.

What Phase 1 is **not**: it is not Auth0, not a backend, not a database, not a real catalog service,
and not a fake backend pretending to be one. Fixtures are typed in-repo data, not a mock server.

---

## 2. Why Phase 1 comes first

Per the architecture's sequencing (`docs/ARCHITECTURE.md` §3.2), the first runnable increment is a
Next.js + TypeScript frontend, before Auth0 and before any backend capability. This lets the
developer learn the browser/server model, component/state model, routing, and accessibility on
safe, data-free ground before security and distributed-systems complexity arrive.

---

## 3. Prerequisite concepts to understand before implementation

These are explained in depth (teach-before-code) at the start of **Sub-phase 1A**:

- **Next.js** — a React framework that adds routing, server/client rendering, and build tooling.
- **Why Next.js** — it is the planned customer/admin frontend and a possible future thin BFF
  boundary; it supports deliberate server vs client rendering by data sensitivity.
- **React** — a library for building UI from composable components with props and state.
- **TypeScript** — JavaScript with static types; catches a class of errors at compile time and
  models UI/API states explicitly. Used because it is the shared language across web and services.
- **package.json** — the project manifest: metadata, scripts, and declared dependencies.
- **package-lock.json** — the exact, resolved dependency tree for reproducible installs.
- **npm** — the Node package manager used to install dependencies and run scripts.
- **dependencies vs devDependencies** — runtime needs vs build/test/lint-time-only needs.
- **Next.js App Router** — the file-system router based on the `app/` directory with layouts,
  nested routes, and server/client component boundaries.
- **ESLint** — a static analysis tool that enforces code quality and catches likely bugs.
- **What a production build validates** — that the app type-checks, lints (per config), compiles,
  and produces an optimized, servable output without runtime-only surprises.

The developer should be able to explain each of these before 1A implementation begins.

---

## 4. Sub-phase plan (1A–1J)

Each sub-phase is implemented **one at a time**, with teach-before-code, explicit approval, tests,
documentation, and a stop. Ordering may be refined via a documented decision, but future-phase
concerns (auth, backend, data stores) must not leak in.

### 1A — Frontend Project Foundation

- **What we will learn:** How a Next.js + TypeScript project is structured and configured; what
  each config/manifest file does; strict TypeScript; linting; and what a production build verifies.
- **Why it matters:** A correct, strict, lint-clean foundation prevents a whole category of defects
  and makes every later sub-phase safer and more legible.
- **NexaCommerce connection:** This is the root of the customer/admin frontend named throughout
  `docs/ARCHITECTURE.md` §4.1.
- **Expected implementation:** Initialize `apps/web` with Next.js (App Router), React, TypeScript
  (strict), Tailwind, ESLint, using npm; add basic project configuration; verify dev server and a
  production build run.
- **Explicitly out of scope:** Auth0, authentication/authorization, any backend, API, database,
  PostgreSQL, Redis, broker, gateway, BFF, Docker, Kubernetes, AWS, CI/CD, product catalog, cart,
  orders, payments, inventory, a fake backend, and any unnecessary dependency.

### 1B — Routing / Application Shell

- **What we will learn:** App Router routing, nested layouts, and a shared application shell
  (header/nav/main/footer) using semantic HTML.
- **Why it matters:** Routing and layout structure are the skeleton every page hangs on; semantic
  structure is the foundation of accessibility.
- **NexaCommerce connection:** The shell hosts customer and (later) minimal admin routes in one web
  app, per `docs/ARCHITECTURE.md` §4.1.
- **Expected implementation:** Define route/layout structure and a basic accessible shell with
  placeholder routes.
- **Explicitly out of scope:** Real data, auth-gated routes, admin authorization, and any backend
  calls.

### 1C — Server vs Client Components

- **What we will learn:** The distinction between server and client components, when each is
  appropriate, and how rendering/caching choices follow from data sensitivity and interactivity.
- **Why it matters:** Choosing the wrong boundary leaks secrets, harms performance, or breaks
  interactivity. `docs/ARCHITECTURE.md` §4.1 requires keeping browser concerns separate from
  server-only concerns.
- **NexaCommerce connection:** Public catalog can use server rendering/caching; personalized data
  (later: cart/profile/checkout/admin) must not leak through shared caches.
- **Expected implementation:** Demonstrate deliberate server vs client component choices on the
  shell/catalog pages.
- **Explicitly out of scope:** Secrets, tokens, protected APIs, and personalized data (no auth yet).

### 1D — Design System / Accessible UI Foundation

- **What we will learn:** Design tokens, consistent styling with Tailwind, and a WCAG 2.2 AA
  accessibility baseline (semantic structure, keyboard operability, visible focus, contrast, labels).
- **Why it matters:** `docs/PRD.md` §10.4 targets WCAG 2.2 AA for customer and admin experiences;
  accessibility and design consistency are cheaper to build in than to retrofit.
- **NexaCommerce connection:** Shared design primitives organized by product feature, not by
  mirroring microservice internals (`docs/ARCHITECTURE.md` §4.1).
- **Expected implementation:** Establish design tokens and a small set of accessible, reusable UI
  primitives.
- **Explicitly out of scope:** A heavyweight component library beyond need, theming systems, or any
  backend-driven styling.

### 1E — Typed Product Fixture Model

- **What we will learn:** Modeling domain data with TypeScript types and representing a product with
  fixtures, honoring the conceptual Product entity in `docs/DATABASE.md`.
- **Why it matters:** Typed fixtures let the UI be built and tested with no backend, while keeping
  the shape honest to the planned contract.
- **NexaCommerce connection:** Mirrors the Product/Category entity outlines (SKU, name, slug, price
  amount/currency, publication status, media references) without implementing the Product Service.
- **Expected implementation:** Define product/category TypeScript types and in-repo fixture data;
  represent money as `{ amount, currency }` (no floating point) per `docs/API.md` §2.1.
- **Explicitly out of scope:** Any database, API call, Product Service, or persistence. Fixtures are
  static in-repo data, not a mock server.

### 1F — Product Catalog

- **What we will learn:** Rendering a list/grid from typed fixtures; basic client-side
  filter/sort patterns; responsive layout.
- **Why it matters:** The catalog is the entry of the customer journey (`docs/PRD.md` §7.1).
- **NexaCommerce connection:** Implements the discover step of discover → cart → checkout → track,
  using fixtures as a stand-in for the future Product Service reads.
- **Expected implementation:** A responsive, accessible catalog page driven by fixtures.
- **Explicitly out of scope:** Server-side search, pagination against a real API, cart actions, and
  any network calls.

### 1G — Product Detail

- **What we will learn:** Dynamic routes and detail pages; presenting price, description, media
  references, and an availability indication from fixtures.
- **Why it matters:** Product detail is where evaluation happens before purchase (`docs/PRD.md`
  FR-C05).
- **NexaCommerce connection:** The detail view a future Product Service `GET /v1/products/{id}` will
  back; for now it reads a fixture.
- **Expected implementation:** A dynamic product-detail route rendering a single fixture product.
- **Explicitly out of scope:** Add-to-cart behavior, reviews, real availability from Inventory, and
  any backend call.

### 1H — UI States / Resilience

- **What we will learn:** Explicit empty, loading, and error states, and how to make them visible
  and accessible.
- **Why it matters:** The Phase 1 Definition of Done requires visible empty/loading/error states;
  resilient UI states are a core React/Next.js skill (`docs/ROADMAP.md` Phase 1).
- **NexaCommerce connection:** Establishes the state-handling pattern reused across cart, checkout,
  and admin later.
- **Expected implementation:** Loading and error boundaries and empty-state handling for catalog and
  detail, driven by fixture scenarios.
- **Explicitly out of scope:** Real network error handling, retries/backoff against a backend, and
  auth error flows.

### 1I — Testing / Accessibility Validation

- **What we will learn:** Component tests for key states, route smoke tests, type/lint checks, and
  keyboard + automated accessibility checks.
- **Why it matters:** Testing begins with the first feature (`docs/LEARNING.md`); the DoD requires
  critical UI to be keyboard usable and key states tested.
- **NexaCommerce connection:** Starts the risk-focused testing discipline carried through every
  phase.
- **Expected implementation:** A test setup (framework chosen with approval) plus tests for catalog,
  detail, and UI states, with at least one normal, one empty/invalid, and one error case; automated
  a11y checks plus manual keyboard verification.
- **Explicitly out of scope:** Backend/integration tests, contract tests, E2E against services, and
  load/security suites (later phases).

### 1J — Documentation / Git Checkpoint

- **What we will learn:** Writing fresh-clone run instructions and preparing a reviewed commit.
- **Why it matters:** The DoD requires fresh-clone instructions to work and decisions to be
  documented; checkpoints are reviewed, never automatic (`docs/KIRO_WORKFLOW.md` §17).
- **NexaCommerce connection:** Produces the `feat(web): establish accessible Next.js storefront
  foundation` checkpoint candidate and updates this learning record from plan to completion status.
- **Expected implementation:** Update README/run docs; update this `PHASE-1.md` to reflect what was
  actually built; propose the commit and wait for approval.
- **Explicitly out of scope:** Pushing without approval, and bundling any Phase 2 work.

---

## 5. Phase 1 Definition of Done (from `docs/ROADMAP.md`)

- Fresh-clone instructions work.
- Empty/loading/error states are visible.
- No secrets or Auth0 placeholders presented as real.
- Critical UI is keyboard usable.
- Decisions are documented.
- Git checkpoint: `feat(web): establish accessible Next.js storefront foundation`.

---

## 6. Deferred / to-be-decided during Phase 1

- Node.js LTS version to target.
- Styling specifics beyond Tailwind baseline, and the test framework/runner choice (decided with
  approval, recorded here or via ADR if it shapes architecture).

---

## 7. Things explicitly NOT in Phase 1

Auth0, authentication/authorization, any backend or API, PostgreSQL/Redis/broker/gateway/BFF,
Docker/Kubernetes/cloud, CI/CD, cart/orders/payments/inventory logic, a fake backend, and any
dependency not required by the approved sub-phase.

---

## 8. Checkpoint status

**NOT STARTED.** Phase 1 has no implementation. This document will be updated sub-phase by
sub-phase as work is actually done, and its status will change from PLANNED to IN PROGRESS to
COMPLETE/CHECKPOINTED only when the repository supports those claims.
