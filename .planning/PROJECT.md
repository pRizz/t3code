# T3 Code Reliability Audit

## What This Is

This initiative treats the existing T3 Code repository as a brownfield system that needs a full, evidence-driven bug audit before more feature work or refactors are prioritized. The goal of this milestone is to inspect the current server, web, desktop, contracts, shared utilities, scripts, and release flows deeply enough to produce a trustworthy catalog of bugs and operational issues for maintainers.

## Core Value

Produce a bug catalog that is accurate enough to drive later remediation phases without guessing about where the real correctness, reliability, security, and UX risks are.

## Requirements

### Validated

- ✓ T3 Code already ships a Codex-first local web GUI backed by a shared server/runtime architecture — existing
- ✓ T3 Code already supports both browser and Electron desktop shells over the same core backend — existing
- ✓ T3 Code already persists orchestration state, provider session state, and git checkpoint metadata locally — existing
- ✓ T3 Code already streams typed orchestration and terminal events between server and UI — existing

### Active

- [ ] Complete a comprehensive codebase audit focused on real bugs, failure modes, and operational issues
- [ ] Build a catalog that distinguishes confirmed bugs from high-confidence likely defects and explicitly marks verification gaps
- [ ] Record evidence, affected files, failure mode, severity, confidence, and fix direction for every finding
- [ ] Use the catalog to shape later bug-fix phases instead of fixing issues ad hoc during initialization

### Out of Scope

- Implementing bug fixes during initialization — this milestone is discovery-first and should stay scoped to audit artifacts
- Re-architecting maintainability issues without a concrete bug or failure risk attached — defer until the catalog drives a specific remediation phase
- Net-new product features unrelated to current defects — keep the milestone centered on correctness, reliability, and UX issues

## Context

- The repo is a Bun/Node TypeScript monorepo with four main surfaces: `apps/server`, `apps/web`, `apps/desktop`, and `apps/marketing`, plus shared contracts/utilities in `packages/`.
- The product is early-stage and already expects bugs, as stated in `README.md`.
- A current codebase map exists in `.planning/codebase/`, including architecture, stack, testing, and known concern summaries.
- Existing maintainability plans in `.plans/` already highlight oversized critical-path files and runtime hardening work, which suggests there is meaningful latent defect risk in coordinator-heavy modules.
- The user explicitly wants the first milestone to be a deep audit and detailed bug catalog; deciding which bugs to fix comes later.

## Constraints

- **Audit scope**: Cover server, web, desktop, contracts, shared utilities, scripts, CI/release, and user-visible download flows — because the bug catalog needs to represent the whole shipped system, not just one app package
- **Evidence quality**: Findings should cite concrete code paths and, where feasible, command/test evidence — because later phases will depend on this catalog for prioritization
- **Execution guardrail**: Do not fix source behavior during initialization — because the immediate goal is to finish discovery and preserve a clean decision point for later remediation phases
- **Project conventions**: Before considering the task complete, `bun run fmt`, `bun run lint`, and `bun run typecheck` must pass — because repo instructions require these checks
- **Testing command**: Never use `bun test`; use `bun run test` when running the workspace test suite — because the repo explicitly forbids `bun test`

## Key Decisions

| Decision                                                                                        | Rationale                                                                                         | Outcome   |
| ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | --------- |
| Treat this as a brownfield audit initiative rather than a feature milestone                     | The repo already exists and the user wants discovery before action                                | — Pending |
| Skip external domain-ecosystem research during initialization                                   | The current milestone is about the repo's actual code and behavior, not market/domain exploration | — Pending |
| Separate confirmed bugs, high-confidence likely bugs, and open verification gaps in the catalog | This avoids overstating certainty while still surfacing likely risks                              | — Pending |
| Defer bug fixes to later phases after the initial catalog is complete                           | The user explicitly wants the initial catalog first and remediation sequencing second             | — Pending |
| Commit planning and audit artifacts to git                                                      | The catalog should remain versioned and reviewable as a planning baseline                         | — Pending |

---

_Last updated: 2026-03-14 after initialization_
