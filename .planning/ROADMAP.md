# Roadmap: T3 Code Reliability Audit

## Overview

This roadmap turns the existing T3 Code repository into an evidence-backed bug discovery project before any remediation work is chosen. The first half of the roadmap focuses on subsystem-by-subsystem audit coverage and evidence quality; the second half turns those findings into a durable catalog, grouped remediation waves, and an execution handoff for later fix phases.

## Phases

**Phase Numbering:**

- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Audit Charter and Scope Rules** - Lock the discovery-only rules and establish how findings will be classified.
- [x] **Phase 2: Server Runtime and Provider Audit** - Audit the server, orchestration, transport, and provider lifecycle paths.
- [x] **Phase 3: Web and Desktop Surface Audit** - Audit user-visible client and desktop flows for correctness and UX issues.
- [x] **Phase 4: Data, Git, Tooling, and Release Audit** - Audit persistence, git/checkpointing, shared utilities, scripts, CI, and release/download paths.
- [x] **Phase 5: Evidence Validation and Gap Sweep** - Verify findings where feasible and document remaining blind spots honestly.
- [x] **Phase 6: Initial Bug Catalog Assembly** - Produce the canonical catalog of findings with consistent metadata.
- [x] **Phase 7: Remediation Wave Design** - Group findings into coherent later fix waves based on severity and subsystem.
- [x] **Phase 8: Fix-Phase Sequencing and Handoff** - Sequence later implementation phases once the catalog is complete.

## Phase Details

### Phase 1: Audit Charter and Scope Rules

**Goal**: Define the audit methodology, discovery-only scope, and finding taxonomy before subsystem review begins.
**Depends on**: Nothing (first phase)
**Requirements**: SCOP-01, SCOP-02
**Success Criteria** (what must be TRUE):

1. Audit artifacts clearly separate confirmed bugs, likely bugs, and open verification gaps.
2. Initialization and cataloging work do not change source behavior.
3. Maintainers can tell what the audit covers and what it intentionally does not cover.
   **Plans**: 2 plans

Plans:

- [x] 01-01: Define audit scope, evidence standards, and classification rules
- [x] 01-02: Create discovery-only guardrails and artifact structure

### Phase 2: Server Runtime and Provider Audit

**Goal**: Audit the server-side runtime paths that can lose work, corrupt state, mis-sequence events, or expose unsafe behavior.
**Depends on**: Phase 1
**Requirements**: SERV-01, SERV-02
**Success Criteria** (what must be TRUE):

1. Provider session lifecycle and turn orchestration risks are cataloged with code evidence.
2. WebSocket request, push, startup, and shutdown flows are reviewed for correctness and failure handling issues.
3. Findings identify concrete files and likely user-visible impact.
   **Plans**: 3 plans

Plans:

- [x] 02-01: Audit provider session, turn, and ingestion flows
- [x] 02-02: Audit orchestration, push bus, and readiness sequencing
- [x] 02-03: Validate server findings against tests and runtime commands where feasible

### Phase 3: Web and Desktop Surface Audit

**Goal**: Audit the client surfaces that users directly experience, including browser and Electron-specific behavior.
**Depends on**: Phase 2
**Requirements**: CLNT-01, CLNT-02
**Success Criteria** (what must be TRUE):

1. Chat/session UX and client state bugs are cataloged with concrete reproduction paths or hypotheses.
2. Desktop bridge, update, and packaging-adjacent issues are identified with operational impact.
3. User-visible bugs are separated from purely maintainability concerns.
   **Plans**: 3 plans

Plans:

- [x] 03-01: Audit web transport, store, routes, and chat UX logic
- [x] 03-02: Audit desktop bridge, lifecycle, and update behavior
- [x] 03-03: Validate client findings against browser/desktop-adjacent checks where feasible

### Phase 4: Data, Git, Tooling, and Release Audit

**Goal**: Audit data integrity, repository workflows, and operational tooling paths that can fail silently or break releases.
**Depends on**: Phase 3
**Requirements**: DATA-01, TOOL-01
**Success Criteria** (what must be TRUE):

1. Persistence, checkpointing, attachments, and git/worktree flows are audited for recovery and integrity issues.
2. Shared utilities, scripts, CI, release, and marketing/download logic are audited for operational bugs.
3. Findings include both product-facing and maintainers-facing impact where relevant.
   **Plans**: 3 plans

Plans:

- [x] 04-01: Audit persistence, checkpointing, attachments, and git flows
- [x] 04-02: Audit shared utilities and repo scripts
- [x] 04-03: Audit CI, release, and download/update surfaces

### Phase 5: Evidence Validation and Gap Sweep

**Goal**: Distinguish well-supported findings from weaker hypotheses and record the blind spots that remain.
**Depends on**: Phase 4
**Requirements**: QUAL-01, QUAL-02
**Success Criteria** (what must be TRUE):

1. Findings are backed by source or command/test evidence where feasible.
2. Unverified hypotheses are clearly labeled instead of being presented as confirmed.
3. Blind spots and missing test coverage are recorded for follow-up.
   **Plans**: 2 plans

Plans:

- [x] 05-01: Run targeted validation commands and tie results to findings
- [x] 05-02: Record unresolved hypotheses and coverage gaps

### Phase 6: Initial Bug Catalog Assembly

**Goal**: Produce the canonical bug catalog in a format that is easy to review and easy to execute against later.
**Depends on**: Phase 5
**Requirements**: CATL-01
**Success Criteria** (what must be TRUE):

1. Every bug entry includes severity, confidence, affected files, failure mode, evidence, and fix direction.
2. The catalog is understandable without rereading the entire raw audit trail.
3. Maintainers can scan the catalog and immediately see the highest-risk issues.
   **Plans**: 2 plans

Plans:

- [x] 06-01: Normalize findings into the canonical catalog format
- [x] 06-02: Review the catalog for consistency and missing metadata

### Phase 7: Remediation Wave Design

**Goal**: Convert the raw catalog into coherent later fix waves rather than an unstructured backlog.
**Depends on**: Phase 6
**Requirements**: CATL-02
**Success Criteria** (what must be TRUE):

1. Findings are grouped into sensible remediation waves by severity, subsystem, and coupling.
2. The catalog calls out which issues should be tackled first and why.
3. Remaining verification work is clearly separated from fix work.
   **Plans**: 2 plans

Plans:

- [x] 07-01: Group bugs into remediation clusters
- [x] 07-02: Attach phase-oriented priority rationale to each cluster

### Phase 8: Fix-Phase Sequencing and Handoff

**Goal**: Define the post-catalog execution sequence for actually fixing the discovered bugs.
**Depends on**: Phase 7
**Requirements**: NEXT-01
**Success Criteria** (what must be TRUE):

1. Later fix phases are ordered only after the initial catalog exists.
2. Maintainers can move from catalog review into planning the first fix wave without redoing discovery.
3. The audit milestone ends with a clean handoff to remediation planning.
   **Plans**: 2 plans

Plans:

- [x] 08-01: Define the first remediation phase candidates
- [x] 08-02: Create execution handoff from catalog to fix planning

### Phase 9: Implement the BUG-001 Fix and Produce an Upstream PR Report

**Goal**: Implement the `projects.writeFile` symlink-escape fix and package the evidence into an upstream-ready PR report that proves the defect, the remediation, and the value of merging it.
**Depends on**: Phase 8
**Requirements**: FIX-01, FIX-02
**Success Criteria** (what must be TRUE):

1. `projects.writeFile` rejects symlink-assisted escape attempts before mutating anything outside the workspace root.
2. Regression coverage proves both the allowed in-workspace write path and the rejected symlink-escape path.
3. Phase artifacts include an upstream-ready PR report with deterministic reproduction steps, impact summary, fix explanation, and post-fix verification evidence.

**Plans**: 2 plans

Plans:

- [x] 09-01: Implement canonical workspace write-path enforcement and regression tests
- [x] 09-02: Produce BUG-001 reproduction evidence and upstream PR report

**Details:**
Deliver the server-side fix for `BUG-001`, add regression coverage around workspace-boundary enforcement, and create a comprehensive PR report that shows the vulnerable setup, exact triggering steps, expected versus actual behavior, the implemented fix, and post-fix validation for a clear upstream pull request.

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9

Phases 1-8 were satisfied by the initial audit, catalog, and planning artifact set created on 2026-03-14. Phase 9 executed the first concrete remediation/reporting follow-up and closes this milestone.

| Phase                                    | Plans Complete | Status   | Completed  |
| ---------------------------------------- | -------------- | -------- | ---------- |
| 1. Audit Charter and Scope Rules         | 2/2            | Complete | 2026-03-14 |
| 2. Server Runtime and Provider Audit     | 3/3            | Complete | 2026-03-14 |
| 3. Web and Desktop Surface Audit         | 3/3            | Complete | 2026-03-14 |
| 4. Data, Git, Tooling, and Release Audit | 3/3            | Complete | 2026-03-14 |
| 5. Evidence Validation and Gap Sweep     | 2/2            | Complete | 2026-03-14 |
| 6. Initial Bug Catalog Assembly          | 2/2            | Complete | 2026-03-14 |
| 7. Remediation Wave Design               | 2/2            | Complete | 2026-03-14 |
| 8. Fix-Phase Sequencing and Handoff      | 2/2            | Complete | 2026-03-14 |
| 9. Implement BUG-001 Fix and PR Report   | 2/2            | Complete | 2026-03-14 |
