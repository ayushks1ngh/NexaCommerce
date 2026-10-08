# NexaCommerce Kiro Operating Contract

- **Document status:** ACTIVE operating contract
- **Audience:** Any Kiro CLI session (and any human reviewer) working on this repository
- **Last reviewed:** 2026-10-08

> **The repository documentation is the source of truth. Previous chat history is not.**
>
> **Kiro must not assume that a previous session's intended work was completed. Verify the repository.**

This document is a persistent operating contract. It survives across Kiro sessions, chat
resets, and context loss. It defines how work on NexaCommerce is conducted so that any new
session can resume safely without relying on memory of earlier conversations.

---

## 1. Project purpose

NexaCommerce is a **learning-focused, production-oriented** commerce platform. The goal is not
to generate a finished application as fast as possible; it is to let the developer **understand**
the engineering concepts behind each capability while building a system that could evolve safely
toward production.

- "Learning-focused" means every increment is designed, explained, tested, and documented, and
  the developer must be able to explain it without reading generated code line by line.
- "Production-oriented" describes design intent and engineering discipline. It is **not** a claim
  that the system is production-ready at any point before an explicit readiness review (Roadmap
  Phase 16).

The authoritative product definition lives in `docs/PRD.md`. If this section and the PRD ever
disagree, the PRD wins and this document must be corrected.

---

## 2. Source-of-truth hierarchy

When information conflicts, resolve in this order (highest authority first):

1. **The repository working tree and git history** — what actually exists and is committed.
2. **Accepted ADRs** (`docs/DECISIONS/ADR-*.md`) — recorded, dated architectural decisions.
3. **`docs/ARCHITECTURE.md`** — system design and status vocabulary.
4. **`docs/PRD.md`** — product requirements and scope.
5. **`docs/API.md` and `docs/DATABASE.md`** — contract and data baselines.
6. **`docs/ROADMAP.md`** — phase sequencing and Definitions of Done.
7. **`docs/LEARNING.md` and `docs/LEARNING/`** — curriculum and phase learning records.
8. **This document (`docs/KIRO_WORKFLOW.md`)** — how to operate.

Chat history, prior session summaries, and any assistant's memory are **not** sources of truth.
They are, at best, hints to be verified against the repository.

---

## 3. Human developer responsibilities

- Owns direction, approvals, and all irreversible decisions (commits, pushes, dependency
  additions, architecture changes).
- Chooses when to advance from one sub-phase to the next.
- Approves or rejects each proposed implementation plan before any code is written.
- Confirms understanding during the teach-back step; may ask Kiro to re-explain.
- Decides open product/architecture questions when a phase requires them.
- Reviews diffs before any checkpoint.

---

## 4. Kiro responsibilities

- Recover and report project state at the start of every session (see Section 5).
- Read the relevant documentation before proposing or writing anything.
- Teach the concepts for the current sub-phase **before** implementing (Section 9).
- Propose a precise, minimal implementation plan and **wait for explicit approval**.
- Implement only the approved sub-phase, nothing more.
- Validate the work (build/type/lint/tests as applicable) and report honest results.
- Explain the important code and configuration produced.
- Update the learning and project documentation to match reality.
- Stop and wait after each sub-phase; never auto-advance, auto-commit, or auto-push.
- Flag any discrepancy between documentation and repository rather than silently "fixing" it.

---

## 5. Session recovery protocol

Every new Kiro session begins read-only. Before proposing any change, Kiro must:

1. Run `pwd`, `git status`, `git branch --show-current`, `git log --oneline --decorate -10`.
2. Inspect the repository structure and the `docs/` tree as it actually is.
3. Read the source-of-truth documents (Section 2) relevant to the current work.
4. Read `docs/LEARNING/` for the current and previous phase records.
5. Produce a **Project Recovery Report** covering: current branch, last commit, whether the
   working tree is clean, existing documentation, current phase and sub-phase, completed work,
   in-progress work, not-yet-implemented work, pending decisions, known problems, and the
   proposed next action.
6. **Do not modify anything during recovery.** Verify claims against files; never assume a prior
   session finished what it said it would.

Only after the recovery report, and after the human confirms the next action, does work proceed.

---

## 6. Current phase / sub-phase tracking

- The authoritative current phase is whatever `docs/ROADMAP.md` marks as CURRENT, cross-checked
  against what actually exists in the repository.
- Phase learning records in `docs/LEARNING/PHASE-X.md` carry a **completion/checkpoint status**
  line that states whether the phase is PLANNED, IN PROGRESS, or COMPLETE/CHECKPOINTED.
- Sub-phase granularity (for example Phase 1's 1A–1J) is tracked inside the relevant phase
  learning document. A sub-phase is only "done" when its work exists, is validated, is explained,
  and its documentation is updated.
- As of this document's last review, the project is at **Phase 0 (foundation)**, with
  documentation authored and the Phase 0 git checkpoint still pending.

---

## 7. The workflow: Understand → Design → Implement → Test → Document → Review → Checkpoint

Every sub-phase follows this cycle in order:

1. **Understand** — Read the relevant documentation and the current repository state.
2. **Design** — Explain the concepts and propose a minimal implementation plan. Wait for approval.
3. **Implement** — Build only the approved scope.
4. **Test** — Validate with the appropriate checks (type, lint, build, unit/integration,
   accessibility, failure cases) and report results honestly, including what could not be verified.
5. **Document** — Update the phase learning record and any affected project documentation/ADRs.
6. **Review** — Show changed files and explain the important code/configuration; the human reviews.
7. **Checkpoint** — Propose (never perform automatically) a focused git commit. Wait for approval.

Then **STOP**.

---

## 8. One-sub-phase-at-a-time rule

- Work on exactly one sub-phase at a time.
- Do not bundle multiple sub-phases into a single plan, implementation, or commit.
- Do not pull future-phase technology into an earlier phase for convenience.
- Completing a sub-phase does not grant permission to start the next one.

---

## 9. Teach-before-code rule

- Before implementing a sub-phase, Kiro explains what is being learned, why it matters, and how it
  connects to NexaCommerce's architecture.
- The explanation must be understandable without first reading generated code.
- Implementation proceeds only after the human has had the chance to absorb the explanation and
  approve the plan.
- Code volume is never a substitute for understanding.

---

## 10. No automatic continuation

- Kiro never advances from one sub-phase or phase to the next on its own.
- After reporting results for a sub-phase, Kiro stops and waits for an explicit instruction.
- "The build passed" is not permission to continue.

---

## 11. No automatic commit

- Kiro never runs `git commit` without explicit human approval of that specific commit.
- Kiro proposes commit contents (which files, which message) and waits.
- Commits are focused: one coherent unit of work per commit.
- Never commit secrets, credentials, tokens, or `.env` files.

---

## 12. No automatic push

- Kiro never runs `git push` without explicit human approval.
- Kiro never pushes directly to `main`/`master` unless the human explicitly permits it.
- Branch, PR, and remote operations are proposed, not performed unilaterally.

---

## 13. No silent architectural decisions

- Architectural choices are recorded as ADRs in `docs/DECISIONS/` with context, decision, status,
  and date.
- Kiro must not introduce a framework, library, broker, cache, gateway, container platform, or
  structural pattern without an explicit decision and the human's approval.
- Existing architecture in `docs/ARCHITECTURE.md` and the ADRs is treated as the source of truth
  and must not be rewritten merely because a different approach is preferred. It changes only when
  there is an actual contradiction or a newly required decision — and then via a documented update.

---

## 14. Dependency discipline

- Add a dependency only when a sub-phase genuinely requires it and the human approves.
- Prefer well-known, actively maintained packages; watch for typosquatting look-alikes.
- Pin versions (exact or deliberately constrained), not open-ended ranges.
- Do not add dependencies for convenience, speculative future use, or symmetry.
- Record non-obvious dependency choices in the phase learning record or an ADR when they shape
  architecture.

---

## 15. Testing and validation expectations

- Testing is not a late phase; it begins with the first relevant feature.
- For each implemented behavior, aim to cover at least: one normal case, one invalid/unauthorized
  case, and one relevant failure/concurrency case where applicable.
- Run the project's build/type/lint and the relevant tests after changes; fix failures before
  presenting results.
- If a test framework does not yet exist when first needed, set up the standard choice for the
  ecosystem (with approval) rather than skipping tests.
- State explicitly what was validated and what could not be validated, and why.
- Clean up temporary files created during validation.

---

## 16. Documentation requirements

- Documentation changes with reality, in the same sub-phase as the work it describes.
- Each phase has a learning record in `docs/LEARNING/PHASE-X.md` capturing objective, concepts,
  what was implemented, important files, commands/tests learned, questions the developer should be
  able to answer, mistakes, decisions, intentionally-not-implemented items, and checkpoint status.
- Project documents (`ARCHITECTURE.md`, `API.md`, `DATABASE.md`, `ROADMAP.md`, ADRs) are updated
  when a change makes them inaccurate.
- Proposals must never be documented as if deployed; use the ARCHITECTURE status vocabulary
  (DECIDED / PLANNED / PROPOSED / NOT YET IMPLEMENTED).

---

## 17. Git checkpoint protocol

A checkpoint is a **reviewed commit candidate**, not an automatic action.

1. Confirm the sub-phase/phase meets its Definition of Done in `docs/ROADMAP.md`.
2. Run `git status` and show exactly which files are new/modified/deleted.
3. Present the list of files that **should** be included and those that **should not**.
4. Verify no secrets, credentials, or unintended files are staged.
5. Propose the exact commit message following the roadmap's checkpoint convention
   (for example `docs: establish Phase 0 ...`, `feat(web): ...`).
6. **Wait for explicit human approval** before committing.
7. If the phase is already committed, do not create a duplicate commit.
8. Pushing is a separate, separately-approved step (Section 12).

---

## 18. How a new Kiro session resumes work

1. Treat the repository as the source of truth; ignore assumptions from prior chats.
2. Perform the session recovery protocol (Section 5) and produce the recovery report.
3. Read `docs/KIRO_WORKFLOW.md` (this file), the relevant phase learning record, and the
   source-of-truth documents for the current work.
4. Identify the current phase/sub-phase and its checkpoint status.
5. Propose the next action and wait for approval.
6. Resume the Understand → Design → Implement → Test → Document → Review → Checkpoint cycle.

---

## 19. Handling blocked or incomplete work

- If a sub-phase is incomplete, record its true state in the phase learning document (what is
  done, what remains, what is uncommitted). Do not present it as complete.
- If work is **blocked** (missing decision, missing credential, environment constraint, failing
  dependency), stop and report: what is blocked, why, what is needed to unblock, and any safe
  non-destructive alternatives.
- Never work around a block by silently dropping a requirement, changing architecture, or
  committing partial work as if finished.
- If an approach fails repeatedly, diagnose the root cause and propose a different approach rather
  than making incremental patches; confirm with the human before deviating from the original
  intent.

---

## 20. Scope guardrails

- Implement only what the current approved sub-phase requires. No extra features, abstractions, or
  defensive scaffolding beyond the task.
- Do not let future phases (auth, backend, databases, Redis, broker, gateway, Docker, CI/CD,
  cloud, observability tooling) leak into an earlier phase without a documented need and approval.
- Do not build a fake backend or mock infrastructure to appear further along than the project is.
- Reversibility guides caution: local, reversible edits may proceed; destructive or
  shared-system-affecting actions require explicit approval.
- When in doubt about scope, ask before acting.

---

## 21. Relationship to other documents

- `docs/PRD.md` — what we are building and why (product source of truth).
- `docs/ARCHITECTURE.md` — how the system is designed and its status vocabulary.
- `docs/ROADMAP.md` — the ordered phases and their Definitions of Done.
- `docs/API.md`, `docs/DATABASE.md` — contract and data baselines.
- `docs/DECISIONS/ADR-*.md` — recorded architectural decisions.
- `docs/LEARNING.md` — the overall learning curriculum.
- `docs/LEARNING/` — per-phase learning records/plans.
- `docs/KIRO_WORKFLOW.md` — **this** operating contract.

If this document ever contradicts an accepted ADR or `ARCHITECTURE.md`, those win and this
document must be corrected.
