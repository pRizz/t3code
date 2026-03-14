# Roadmap: T3 Code Reliability Audit

## Overview

This roadmap turns the existing T3 Code repository into an evidence-backed bug discovery project before any remediation work is chosen. The first half of the roadmap focuses on subsystem-by-subsystem audit coverage and evidence quality; the second half turns those findings into a durable catalog, grouped remediation waves, and an execution handoff for later fix phases.

## Phases

**Phase Numbering:**

- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Audit Charter and Scope Rules** - Lock the discovery-only rules and establish how findings will be classified.
- [ ] **Phase 2: Server Runtime and Provider Audit** - Audit the server, orchestration, transport, and provider lifecycle paths.
- [ ] **Phase 3: Web and Desktop Surface Audit** - Audit user-visible client and desktop flows for correctness and UX issues.
- [ ] **Phase 4: Data, Git, Tooling, and Release Audit** - Audit persistence, git/checkpointing, shared utilities, scripts, CI, and release/download paths.
- [ ] **Phase 5: Evidence Validation and Gap Sweep** - Verify findings where feasible and document remaining blind spots honestly.
- [ ] **Phase 6: Initial Bug Catalog Assembly** - Produce the canonical catalog of findings with consistent metadata.
- [ ] **Phase 7: Remediation Wave Design** - Group findings into coherent later fix waves based on severity and subsystem.
- [ ] **Phase 8: Fix-Phase Sequencing and Handoff** - Sequence later implementation phases once the catalog is complete.

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

- [ ] 01-01: Define audit scope, evidence standards, and classification rules
- [ ] 01-02: Create discovery-only guardrails and artifact structure

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

- [ ] 02-01: Audit provider session, turn, and ingestion flows
- [ ] 02-02: Audit orchestration, push bus, and readiness sequencing
- [ ] 02-03: Validate server findings against tests and runtime commands where feasible

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

- [ ] 03-01: Audit web transport, store, routes, and chat UX logic
- [ ] 03-02: Audit desktop bridge, lifecycle, and update behavior
- [ ] 03-03: Validate client findings against browser/desktop-adjacent checks where feasible

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

- [ ] 04-01: Audit persistence, checkpointing, attachments, and git flows
- [ ] 04-02: Audit shared utilities and repo scripts
- [ ] 04-03: Audit CI, release, and download/update surfaces

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

- [ ] 05-01: Run targeted validation commands and tie results to findings
- [ ] 05-02: Record unresolved hypotheses and coverage gaps

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

- [ ] 06-01: Normalize findings into the canonical catalog format
- [ ] 06-02: Review the catalog for consistency and missing metadata

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

- [ ] 07-01: Group bugs into remediation clusters
- [ ] 07-02: Attach phase-oriented priority rationale to each cluster

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

- [ ] 08-01: Define the first remediation phase candidates
- [ ] 08-02: Create execution handoff from catalog to fix planning

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8

| Phase                                    | Plans Complete | Status      | Completed |
| ---------------------------------------- | -------------- | ----------- | --------- |
| 1. Audit Charter and Scope Rules         | 0/2            | Not started | -         |
| 2. Server Runtime and Provider Audit     | 0/3            | Not started | -         |
| 3. Web and Desktop Surface Audit         | 0/3            | Not started | -         |
| 4. Data, Git, Tooling, and Release Audit | 0/3            | Not started | -         |
| 5. Evidence Validation and Gap Sweep     | 0/2            | Not started | -         |
| 6. Initial Bug Catalog Assembly          | 0/2            | Not started | -         |
| 7. Remediation Wave Design               | 0/2            | Not started | -         |
| 8. Fix-Phase Sequencing and Handoff      | 0/2            | Not started | -         |
