---
phase: 09-implement-the-bug-001-fix-and-produce-an-upstream-pr-report
plan: "01"
subsystem: api
tags: [websocket, filesystem, path-safety, symlink, vitest]
requires: []
provides:
  - Canonical workspace-boundary enforcement for `projects.writeFile`
  - Regression coverage for allowed in-workspace writes and rejected symlink escapes
  - Verification evidence for the BUG-001 server remediation
affects: [09-02, upstream-pr-report, server-security]
tech-stack:
  added: []
  patterns:
    - Canonicalize the deepest existing ancestor before mutating a requested workspace path
    - Preserve lexical fast-fail checks before the canonical filesystem boundary check
key-files:
  created: []
  modified:
    - apps/server/src/wsServer.ts
    - apps/server/src/wsServer.test.ts
key-decisions:
  - "Kept the existing lexical path rejection and added canonical-path validation before directory creation or file writes."
  - "Resolved missing target files through the deepest existing ancestor so new-file writes stay safe without requiring the leaf file to exist first."
patterns-established:
  - "Workspace write routes must compare canonical root and target paths before any filesystem mutation."
  - "Route-level security fixes need regression tests that prove both rejection and non-mutation."
requirements-completed:
  - FIX-01
  - FIX-02
duration: 21min
completed: 2026-03-14
---

# Phase 9 Plan 01: Canonical Workspace Write-Path Enforcement Summary

**Canonical workspace-boundary enforcement for `projects.writeFile`, backed by a symlink-escape regression and full repo verification**

## Performance

- **Duration:** 21 min
- **Started:** 2026-03-14T10:29:00Z
- **Completed:** 2026-03-14T10:50:00Z
- **Tasks:** 3
- **Files modified:** 2

## Accomplishments

- Hardened `resolveWorkspaceWritePath()` so the route canonicalizes both the workspace root and the requested target before any filesystem mutation.
- Preserved ordinary in-workspace writes while rejecting both lexical escapes and symlink-assisted escapes through existing ancestors.
- Added a deterministic WebSocket-route regression that proves the outside file is never created when a workspace symlink points out of tree.

## Task Commits

- **Implementation commit:** `6cff3f2` (`fix(09-01): harden workspace write path resolution`)
- **Execution note:** Tasks 1-3 landed in a single implementation commit because the route hardening, regression, and verification were tightly coupled.
- **Plan metadata:** Pending the phase-completion documentation commit.

## Files Created/Modified

- `apps/server/src/wsServer.ts` - Added canonical ancestor resolution plus root/target boundary checks for `projects.writeFile`.
- `apps/server/src/wsServer.test.ts` - Added the symlink-escape regression while keeping the happy-path and lexical-reject coverage green.

## Decisions Made

- Kept the lexical guard as the first fast-fail so obviously invalid paths still return simple, stable route errors.
- Used the deepest existing ancestor plus `realpathSync.native()` so new-file writes can be validated safely before the leaf exists.
- Returned the canonical target path to the write route so downstream directory creation and file writes cannot bypass the earlier boundary check.

## Deviations from Plan

### Auto-fixed Issues

**1. [Typecheck cleanup] `Effect.gen` branches needed yielded error values instead of `Effect.fail(...)` wrappers**

- **Found during:** Task 3 (repo verification for the hardened route)
- **Issue:** The first pass used `yield* Effect.fail(new RouteRequestError(...))`, which triggered `TS29` diagnostics during `bun run typecheck`.
- **Fix:** Replaced those branches with direct `return yield* new RouteRequestError(...)` yields inside `resolveWorkspaceWritePath()`.
- **Files modified:** `apps/server/src/wsServer.ts`
- **Verification:** `bun run typecheck`, `bun run build`, and `bun run test`
- **Committed in:** `6cff3f2`

---

**Total deviations:** 1 auto-fixed typecheck issue
**Impact on plan:** No scope change. The cleanup was required to keep the intended fix valid under the repo's standard verification gate.

## Issues Encountered

None beyond the typecheck cleanup captured above.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- The BUG-001 fix now has real implementation and regression evidence for the upstream report to cite directly.
- The next plan can focus on maintainer-facing reproduction/reporting without making further runtime changes.

---

_Phase: 09-implement-the-bug-001-fix-and-produce-an-upstream-pr-report_
_Completed: 2026-03-14_
