# NexaCommerce Learning Records

- **Document status:** ACTIVE
- **Last reviewed:** 2026-10-08

This directory holds the **detailed, per-phase learning records** for NexaCommerce. It works
together with the top-level curriculum but serves a different purpose.

## Two complementary documents

| Document | Purpose | Scope |
|---|---|---|
| `docs/LEARNING.md` | The **overall learning curriculum** — the full map of topics (Git, web, HTTP, TypeScript, React, Next.js, auth, OAuth/OIDC/JWT, Auth0, REST, microservices, PostgreSQL, Redis, distributed systems, brokers, events, Docker, testing, CI/CD, cloud, observability, security) and when each is encountered. | Cross-phase, stable, high-level |
| `docs/LEARNING/PHASE-X.md` | The **detailed learning record or plan for one phase** — what that phase teaches, what was actually implemented (or is planned), important files, commands and tests learned, decisions, mistakes, and the questions the developer should be able to answer. | One phase, evolving with reality |

In short:

- **`LEARNING.md` = overall curriculum** (the syllabus).
- **`LEARNING/PHASE-X.md` = detailed phase learning record/plan** (the lab notebook for that phase).

`docs/LEARNING.md` is **not** replaced by this directory. It remains the authoritative curriculum.
These phase files reference it and add depth.

## What each phase record contains

A completed-phase record captures what was *actually* learned and built. A not-yet-started phase
record is a **plan**, clearly marked PLANNED / NOT YET IMPLEMENTED, describing the intended
journey. Typical contents:

- Phase objective and learning objective
- Concepts to understand and why they matter
- How the concepts connect to NexaCommerce's architecture
- What should be understood **before** implementation
- What was implemented (or is planned)
- Important files
- Commands learned
- Testing/validation learned
- Questions the developer should be able to answer
- Common mistakes
- Architectural decisions
- Things intentionally **not** implemented
- Completion / checkpoint status

## Honesty rule

A phase record must reflect the real state of the repository. It must not claim the developer
learned something, or that something was implemented, when the repository and documentation do
not support that claim. Plans are labelled as plans; completed work is labelled as completed only
after it is validated and checkpointed.

## Current files

- `README.md` — this file.
- `PHASE-0.md` — Product and architecture foundation (learning record).
- `PHASE-1.md` — Next.js frontend foundation (learning **plan**, not yet implemented).

Future phase files (`PHASE-2.md` onward) are created **when that phase begins**, not in advance.
Empty placeholder files are intentionally avoided.
