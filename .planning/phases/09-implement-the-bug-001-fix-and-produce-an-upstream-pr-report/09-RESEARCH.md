# Phase 9 Research: BUG-001 Workspace Write Symlink Escape

## Goal

Plan a tight remediation for `BUG-001` that:

1. closes the `projects.writeFile` symlink escape in `apps/server/src/wsServer.ts`,
2. proves the fix with durable regression coverage, and
3. leaves behind an upstream-ready report that makes the reproduction and value obvious.

## Current Implementation Snapshot

### Vulnerable path resolution

- `resolveWorkspaceWritePath()` in [apps/server/src/wsServer.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.ts#L160) only uses lexical `path.resolve()` plus `path.relative()` checks.
- The function returns `absolutePath` directly for later filesystem mutation without any canonical-path or symlink inspection.

### Write path mutation happens immediately after lexical validation

- The `projects.writeFile` route in [apps/server/src/wsServer.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.ts#L753) calls `makeDirectory(dirname(target.absolutePath))` and `writeFileString(target.absolutePath, body.contents)` immediately after the lexical guard.
- Both operations follow symlinks on disk, so a path that is lexically inside the workspace can still write outside the workspace if one segment is a symlink.

### Existing tests only cover the lexical cases

- [apps/server/src/wsServer.test.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.test.ts#L1576) proves the happy path for an in-root write.
- [apps/server/src/wsServer.test.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.test.ts#L1601) only rejects a simple `../escape.md` path. There is no symlink traversal test today.

## Root Cause

The route validates the user-supplied path as a string, not as the path the kernel will actually follow. That means the check answers “does this string stay under the workspace?” instead of “where will this write land on disk?”

This is why the bug exists:

1. `path.resolve(workspaceRoot, relativePath)` happily returns a lexically in-root path such as `/repo/link/out.md`.
2. `path.relative(workspaceRoot, absolutePath)` also stays in-root, so validation passes.
3. If `link` is a symlink to `/tmp/outside`, the later directory creation and write follow the symlink and mutate `/tmp/outside/out.md`.

## Constraints For A Safe Fix

### The route must fail before any mutation outside the workspace

The check cannot happen after `makeDirectory()` because that call already follows symlinks. Validation has to happen before any directory creation or file write.

### The target file may not exist yet

`realpath(targetFile)` cannot be the only check because a new file path will not resolve until after creation. The fix has to reason about the workspace root plus the deepest existing ancestor of the requested target path.

### The workspace root itself may need canonicalization

Comparisons should be against the canonical workspace root, not the raw `cwd` string, otherwise a symlinked project root or mixed path aliasing can produce false negatives or inconsistent comparisons.

### The plan should prefer a focused, mergeable change

The upstream project explicitly prefers small bug-fix PRs in [.github/pull_request_template.md](/Users/peterryszkiewicz/Repos/t3code/.github/pull_request_template.md). The remediation should stay scoped to the write route, its tests, and the supporting report artifact.

## Recommended Fix Strategy

### Keep the current lexical guard as the first fast-fail

The existing absolute-path and `..` rejection is still useful. It rejects obviously invalid input early and keeps error messages straightforward.

### Add canonical ancestor validation before filesystem writes

Recommended implementation shape:

1. Canonicalize the workspace root with `realpathSync.native()` or the Effect filesystem equivalent.
2. Resolve the requested target lexically under the workspace root.
3. Walk upward from the requested target until reaching the deepest existing ancestor.
4. Canonicalize that existing ancestor and verify it still lives under the canonical workspace root.
5. Build the final write path from the canonical ancestor plus the unresolved suffix, then create directories and write using that canonical path.

This approach closes the escape even when the leaf file does not exist yet, because the first existing symlinked ancestor reveals the escape before mutation.

### Bias toward explicit rejection when an existing path segment is symlinked out of tree

For BUG-001, the critical behavior is rejecting any path whose canonical resolution leaves the workspace. The phase does not need to generalize into broader filesystem virtualization or alias support.

## Regression Test Strategy

### Must-add coverage

1. Keep the current happy-path in-root write test green.
2. Add a symlink-escape regression test that:
   - creates a workspace temp dir,
   - creates a separate outside temp dir,
   - places a symlink inside the workspace that points to the outside dir,
   - calls `projects.writeFile` through the WebSocket server, and
   - asserts that the route returns an error and no outside file is created.

### Nice-to-have guard if implementation stays simple

- Add a test that confirms a normal nested in-workspace write still works after canonicalization changes.
- If the chosen implementation canonicalizes the workspace root, consider covering a symlinked `cwd` root path only if it can be done without making the patch noisy.

### Verification commands

- Targeted server regression: `bun run --cwd apps/server test src/wsServer.test.ts`
- Full repo verification after the fix: `bun run fmt`, `bun run lint`, `bun run typecheck`, `bun run test`

## Upstream PR Report Requirements

The report should not just describe the code change. It needs to make the bug easy to understand and verify for an upstream maintainer skimming a small fix PR.

Recommended sections:

1. `What Changed` — concise summary of the path-resolution hardening and new regression test coverage.
2. `Why` — explain the security impact: a lexically in-root path can still write outside the workspace through a symlink.
3. `Steps to reproduce` — deterministic workspace setup, symlink creation, request payload, and observed outside write behavior on the vulnerable revision.
4. `Expected vs actual` — short before/after table.
5. `Validation` — exact test commands plus the specific regression test names added by the fix.

The repo’s bug-report template in [.github/ISSUE_TEMPLATE/bug_report.yml](/Users/peterryszkiewicz/Repos/t3code/.github/ISSUE_TEMPLATE/bug_report.yml) is a good model for the reproduction sections: deterministic steps, expected behavior, actual behavior, impact, and environment.

## Recommended Plan Split

### Wave 1

- `09-01`: implement the server-side fix and land regression coverage.

### Wave 2

- `09-02`: write the upstream PR report after the fix exists so the report can cite the actual code path, test names, and verification results.

This keeps the execution order honest: first make the bug impossible, then document the reproduction and the value of the exact fix that landed.
