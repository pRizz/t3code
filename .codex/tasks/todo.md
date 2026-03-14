# Reliability Audit Task

- [x] Review the existing codebase map and new `.planning/` project artifacts to set audit scope.
- [x] Run baseline verification commands for current behavior: `bun run test`, `bun run --cwd apps/web test:browser`, and `bun run build`.
- [x] Audit server runtime, transport, auth, path-safety, and write-route behavior for concrete defects.
- [x] Audit web transport, routing, project/thread ordering, and sidebar helper behavior for user-visible or reliability defects.
- [x] Audit marketing/download flow and desktop/build tooling issues surfaced by source inspection and build output.
- [x] Write `.planning/BUG_CATALOG.md` with confirmed bugs, high-confidence likely defects, verification gaps, and later-cluster hints.
- [ ] Decide remediation sequencing and convert the highest-priority clusters into later execution phases.

## Verification

- `bun run test`
- `bun run --cwd apps/web test:browser`
- `bun run build`
- `bun run fmt`
- `bun run lint`
- `bun run typecheck`
- `git diff --stat`

## Completion Review

- Created `.planning/BUG_CATALOG.md` as the initial audit baseline without changing runtime source behavior.
- The catalog separates confirmed defects, high-confidence likely defects, and open verification gaps so later phases can prioritize from evidence instead of guesses.
- Residual risk: live Codex resume paths, packaged desktop update installs, and a few repo-tracked UX issues still need targeted reproduction before they should be called fully confirmed.
