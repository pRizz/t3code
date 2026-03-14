# BUG-001 Upstream PR Report

This document is the reviewer-facing companion to the BUG-001 fix. It cites the landed implementation and verification evidence from [09-01-SUMMARY](./09-01-SUMMARY.md) so an upstream maintainer can see the vulnerable behavior, the fix, and the validation without rereading the full audit catalog.

## What Changed

- Hardened `projects.writeFile` in `apps/server/src/wsServer.ts` so the route canonicalizes the workspace root and the requested target path before any directory creation or file write.
- Added `resolveCanonicalPath()` and `isPathWithinRoot()` to validate the deepest existing ancestor of the requested target, which closes the escape even when the leaf file does not exist yet.
- Kept the existing lexical fast-fail for absolute paths and `..` escapes so normal route behavior and error messages stay stable.
- Added a focused regression in `apps/server/src/wsServer.test.ts` that creates a symlink inside the workspace pointing outside the tree and proves the route now rejects that write before any outside mutation occurs.

## Why

Before this patch, `resolveWorkspaceWritePath()` only answered a lexical question: "does the user-supplied string stay under the workspace root?" That is not enough for filesystem safety because the kernel follows symlinks on disk, not the input string.

With the old behavior, a client could send a relative path like `linked-outside/escape.md`. If `linked-outside` was a symlink inside the workspace that pointed to another directory outside the workspace, the route would accept the request and then `makeDirectory()` / `writeFileString()` would follow the symlink and mutate the outside location.

That made BUG-001 a real workspace-boundary violation rather than a theoretical path-normalization issue.

## Steps to reproduce

### Preconditions

- Start from the vulnerable revision: `b0cd5cd` (`docs: plan bug-001 remediation`) plus the pre-fix `apps/server/src/wsServer.ts` behavior that existed before `6cff3f2`.
- Install dependencies with `bun install --frozen-lockfile`.
- Symlink creation must be allowed on the host OS. The landed regression uses `"junction"` on Windows and `"dir"` elsewhere.

### Minimal reproduction path

1. Copy the focused regression from this fix into `apps/server/src/wsServer.test.ts`, or check out the fixed branch and temporarily reset only `apps/server/src/wsServer.ts` back to the vulnerable version.
2. Run the targeted server test:

```sh
bun run --cwd apps/server test src/wsServer.test.ts -t "rejects projects.writeFile paths that escape through a symlinked directory"
```

3. The reproduction creates this workspace layout:

```text
<workspace>/
  linked-outside -> <outsideDir>/
<outsideDir>/
```

4. The request payload sent through the WebSocket route is:

```json
{
  "cwd": "<workspace>",
  "relativePath": "linked-outside/escape.md",
  "contents": "# owned\n"
}
```

5. On the vulnerable route implementation, the lexical guard passes because `linked-outside/escape.md` appears to stay inside `<workspace>`.
6. The route then runs `makeDirectory(dirname(target.absolutePath))` and `writeFileString(target.absolutePath, body.contents)` against a path that traverses the symlink.
7. The actual mutation lands at `<outsideDir>/escape.md`, which proves the write escaped the workspace boundary.

### Observed vulnerable outcome

- `response.result` resolves successfully with `{"relativePath":"linked-outside/escape.md"}`
- `response.error` is absent
- `<outsideDir>/escape.md` exists and contains `# owned`

## Expected vs actual

| Scenario                                    | Expected behavior                                                                | Actual behavior before fix                              | Actual behavior after fix                                                                                      |
| ------------------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Symlinked ancestor points outside workspace | Reject the request before any mutation and leave the outside directory untouched | Request succeeds and writes to `<outsideDir>/escape.md` | Request returns `Workspace file path must stay within the project root.` and the outside file is never created |
| Ordinary nested in-workspace write          | Accept the request and return the normalized relative path                       | Accepts the request                                     | Still accepts the request and returns the same `relativePath`                                                  |

## Fix summary

- `resolveWorkspaceWritePath()` now keeps the lexical guard, then canonicalizes:
  - the workspace root, and
  - the deepest existing ancestor of the requested target path.
- The route compares the canonical target path against the canonical workspace root before any directory creation or file write.
- If canonical resolution leaves the workspace, the route throws `Workspace file path must stay within the project root.` and exits before mutating anything.
- The route still returns the normalized relative path for legitimate in-workspace writes, so the success payload shape did not change.

## Validation

The implementation and exact verification results are captured in [09-01-SUMMARY](./09-01-SUMMARY.md).

### Focused regression evidence

- `bun run --cwd apps/server test src/wsServer.test.ts`
- Added test name: `rejects projects.writeFile paths that escape through a symlinked directory`
- Preserved test name: `supports projects.writeFile within the workspace root`

### Full repo verification

- `bun run fmt`
- `bun run lint`
- `bun run typecheck`
- `bun run build`
- `bun run test`

### Result

- The focused WebSocket-route regression passes on the fixed code.
- Full repo verification passes after the BUG-001 fix lands.
- Known non-failing warnings remain unchanged from the audit baseline: desktop `MODULE_TYPELESS_PACKAGE_JSON`, large web bundle warnings, zustand persist test stderr, and Node experimental SQLite warnings.

## Reviewer notes

- The patch stays small and focused on one server route plus its regression coverage.
- The report intentionally uses the landed regression as the canonical reproduction path so the same steps demonstrate both the old failure mode and the post-fix protection.
- No UI behavior or external API shape changed; the value is entirely in closing the filesystem escape and proving it with durable tests.
