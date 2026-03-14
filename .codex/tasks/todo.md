# Reliability Audit Task

- [x] Review the existing codebase map and new `.planning/` project artifacts to set audit scope.
- [x] Run baseline verification commands for current behavior: `bun run test`, `bun run --cwd apps/web test:browser`, and `bun run build`.
- [x] Audit server runtime, transport, auth, path-safety, and write-route behavior for concrete defects.
- [x] Audit web transport, routing, project/thread ordering, and sidebar helper behavior for user-visible or reliability defects.
- [x] Audit marketing/download flow and desktop/build tooling issues surfaced by source inspection and build output.
- [x] Write `.planning/BUG_CATALOG.md` with confirmed bugs, high-confidence likely defects, verification gaps, and later-cluster hints.
- [x] Decide remediation sequencing and convert the highest-priority clusters into later execution phases.
- [x] Implement BUG-001 remediation and produce an upstream-ready reproduction/PR report.
- [ ] Audit the completed milestone for cross-phase completeness and residual gaps.

## Verification

- `bun run test`
- `bun run --cwd apps/web test:browser`
- `bun run build`
- `bun run --cwd apps/server test src/wsServer.test.ts`
- `rg -n "What Changed|Why|Steps to reproduce|Validation|09-01-SUMMARY" .planning/phases/09-implement-the-bug-001-fix-and-produce-an-upstream-pr-report/09-UPSTREAM-PR-REPORT.md`
- `bun run fmt`
- `bun run lint`
- `bun run typecheck`
- `git diff --stat`

## Completion Review

- Completed the audit milestone artifact set, then closed BUG-001 by hardening `projects.writeFile` against canonical path escapes through symlinked ancestors.
- Added durable server regression coverage and wrote a reviewer-facing reproduction/PR report that ties directly to the landed fix evidence.
- Residual risk: the rest of the cataloged bugs remain open for future remediation, and the known non-failing warning set from build/test still exists.
