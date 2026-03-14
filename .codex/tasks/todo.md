# Codebase Map Task

- [x] Inspect repo manifests, source layout, and existing docs for current architecture signals.
- [x] Gather representative server, web, desktop, contracts, shared, and workflow files.
- [x] Write `.planning/codebase/STACK.md`.
- [x] Write `.planning/codebase/INTEGRATIONS.md`.
- [x] Write `.planning/codebase/ARCHITECTURE.md`.
- [x] Write `.planning/codebase/STRUCTURE.md`.
- [x] Write `.planning/codebase/CONVENTIONS.md`.
- [x] Write `.planning/codebase/TESTING.md`.
- [x] Write `.planning/codebase/CONCERNS.md`.
- [x] Verify all seven codebase docs exist and have meaningful content.
- [x] Run secret scan over `.planning/codebase/*.md`.
- [x] Run `bun run fmt`, `bun run lint`, and `bun run typecheck`.
- [x] Review diff for unintended side effects.

## Verification

- `wc -l .planning/codebase/*.md`
- `grep -E '(sk-[a-zA-Z0-9]{20,}|sk_live_[a-zA-Z0-9]+|sk_test_[a-zA-Z0-9]+|ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|glpat-[a-zA-Z0-9_-]+|AKIA[A-Z0-9]{16}|xox[baprs]-[a-zA-Z0-9-]+|-----BEGIN.*PRIVATE KEY|eyJ[a-zA-Z0-9_-]+\\.eyJ[a-zA-Z0-9_-]+\\.)' .planning/codebase/*.md`
- `bun run fmt`
- `bun run lint`
- `bun run typecheck`
- `git diff --stat`

## Completion Review

- Created the full `.planning/codebase/` map with seven template-aligned documents plus this task record.
- Verification passed: `wc -l .planning/codebase/*.md`, secret scan, `bun run fmt`, `bun run lint`, and `bun run typecheck`.
- Residual risk: the documents are a current-state map, so they will drift unless refreshed after major provider/runtime or UI architecture changes.
