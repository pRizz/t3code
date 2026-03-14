# Bug Catalog: T3 Code Reliability Audit

**Catalog date:** 2026-03-14
**Scope:** `apps/server`, `apps/web`, `apps/desktop`, `apps/marketing`, shared packages, and shipped build/release behavior

## Evidence Base

- Source inspection of the current monorepo, with file/line references for each finding
- Verification commands already run during this audit:
  - `bun run test`
  - `bun run --cwd apps/web test:browser`
  - `bun run build`
- Command-output findings are called out explicitly where the issue is operational rather than source-local

## Classification

- `Confirmed`: direct source or command evidence demonstrates the defect
- `High-confidence likely`: strong source evidence, but the exact user-facing failure path was not exercised live
- `Verification gap`: tracked or suspected issue that still needs targeted reproduction

## Severity Scale

- `P1`: security, data-loss, or severe correctness issue
- `P2`: important reliability or user-flow breakage
- `P3`: moderate UX or operational issue

## Summary

| ID      | Status                 | Severity | Area               | Title                                                                                                         | Suggested Later Cluster                    |
| ------- | ---------------------- | -------- | ------------------ | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| BUG-001 | Confirmed              | P1       | Server security    | `projects.writeFile` can escape the workspace via symlink traversal                                           | Cluster A: Server exposure and path safety |
| BUG-002 | Confirmed              | P1       | Server security    | Auth tokens protect WebSocket upgrades only; attachment and favicon HTTP routes stay public                   | Cluster A: Server exposure and path safety |
| BUG-003 | Confirmed              | P1       | Server runtime     | Standalone web mode can bind publicly without auth or a startup hard-stop                                     | Cluster A: Server exposure and path safety |
| BUG-004 | Confirmed              | P2       | Web transport      | Default WebSocket URL construction breaks on default ports and also poisons favicon origin resolution         | Cluster B: Transport and routing           |
| BUG-005 | Confirmed              | P2       | Web transport      | In-flight WebSocket requests are lost on disconnect but do not reject until the 60s timeout expires           | Cluster B: Transport and routing           |
| BUG-006 | Confirmed              | P2       | Dev server         | Dev-mode HTTP redirects drop the requested path and query string                                              | Cluster B: Transport and routing           |
| BUG-007 | Confirmed              | P2       | Server/web search  | Workspace entry search cache is not invalidated after `projects.writeFile` writes                             | Cluster B: Transport and routing           |
| BUG-008 | Confirmed              | P3       | Marketing/download | macOS hero download detection hardcodes Apple Silicon and misroutes Intel Macs                                | Cluster C: UX and download flows           |
| BUG-009 | Confirmed              | P3       | Web UX             | Newly created projects are appended to the bottom of the project list                                         | Cluster C: UX and download flows           |
| BUG-010 | Confirmed              | P3       | Web UX             | Project ordering ignores latest thread activity and preserves stale manual/persisted order                    | Cluster C: UX and download flows           |
| BUG-011 | High-confidence likely | P3       | Web UX             | Sidebar thread previews sort by creation time, so active older threads can stay buried                        | Cluster C: UX and download flows           |
| BUG-012 | Confirmed              | P3       | Desktop tooling    | Desktop build emits `MODULE_TYPELESS_PACKAGE_JSON` warnings because the package is missing `"type": "module"` | Cluster D: Packaging and build health      |
| BUG-013 | Confirmed              | P2       | Web performance    | Production web build emits oversized chunk warnings, including a roughly 2.5 MB minified main bundle          | Cluster D: Packaging and build health      |

## Detailed Findings

### BUG-001: `projects.writeFile` can escape the workspace via symlink traversal

- Status: `Confirmed`
- Severity: `P1`
- Area: Server security
- Affected files:
  - [apps/server/src/wsServer.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.ts#L160)
- Failure mode:
  - The write route only performs lexical path checks with `path.resolve()` and `path.relative()`.
  - A relative path that stays inside the workspace string-wise but passes through a symlink can still land outside the workspace on disk.
- Evidence:
  - `resolveWorkspaceWritePath()` validates the path before any `realpath` or symlink check and returns an `absolutePath` directly for writing.
  - The `projects.writeFile` route then creates directories and writes to that path immediately.
- Why it matters:
  - A client that can reach this route can overwrite files outside the project root if it targets a symlinked path segment inside the workspace.
- Fix direction:
  - Resolve and compare canonical real paths for the workspace root and target parent.
  - Reject symlink traversal explicitly.
  - Add route tests that cover symlinked directories and files.
- Suggested later cluster: `Cluster A: Server exposure and path safety`

### BUG-002: Auth tokens protect WebSocket upgrades only; attachment and favicon HTTP routes stay public

- Status: `Confirmed`
- Severity: `P1`
- Area: Server security
- Affected files:
  - [apps/server/src/wsServer.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.ts#L425)
  - [apps/server/src/wsServer.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.ts#L932)
- Failure mode:
  - The HTTP server serves `/attachments/*` and `/api/project-favicon` before the WebSocket auth gate is ever reached.
  - The auth token check only runs inside the HTTP upgrade handler.
- Evidence:
  - The HTTP request handler accepts favicon and attachment requests directly.
  - The token check exists only in `httpServer.on("upgrade", ...)`.
- Why it matters:
  - If the server is reachable beyond loopback, an unauthenticated client can fetch attachment content and workspace-identifying favicon responses without the WebSocket token.
- Fix direction:
  - Enforce the same auth policy on sensitive HTTP routes.
  - If public assets must stay open, move sensitive attachment access to signed URLs or authenticated fetches.
- Suggested later cluster: `Cluster A: Server exposure and path safety`

### BUG-003: Standalone web mode can bind publicly without auth or a startup hard-stop

- Status: `Confirmed`
- Severity: `P1`
- Area: Server runtime
- Affected files:
  - [apps/server/src/main.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/main.ts#L160)
  - [apps/server/src/main.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/main.ts#L172)
  - [apps/server/src/main.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/main.ts#L265)
  - [apps/server/src/main.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/main.ts#L298)
- Failure mode:
  - Web mode allows `host` overrides such as `0.0.0.0` or Tailnet IPs while keeping `authToken` optional.
  - Startup logs `authEnabled`, but does not refuse or even warn when the process binds beyond loopback without authentication.
- Evidence:
  - `authToken` is optional during config resolution.
  - `host` can be set freely via CLI/env.
  - Startup proceeds regardless of `host` and `authEnabled` combination.
- Why it matters:
  - The exposed WebSocket surface includes terminal, git, file-write, and orchestration methods; if a user binds publicly without a token, remote clients can drive local tooling.
- Fix direction:
  - Refuse non-loopback binds when auth is missing, or require an explicit `--allow-unauthenticated-public-bind` escape hatch plus a loud warning.
- Suggested later cluster: `Cluster A: Server exposure and path safety`

### BUG-004: Default WebSocket URL construction breaks on default ports and also poisons favicon origin resolution

- Status: `Confirmed`
- Severity: `P2`
- Area: Web transport
- Affected files:
  - [apps/web/src/wsTransport.ts](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/wsTransport.ts#L62)
  - [apps/web/src/components/Sidebar.tsx](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/components/Sidebar.tsx#L185)
- Failure mode:
  - When no desktop bridge URL and no `VITE_WS_URL` are provided, the client constructs `ws://host:${window.location.port}` or `wss://host:${window.location.port}` unconditionally.
  - On standard ports, `window.location.port` is an empty string, so the result becomes `ws://host:` or `wss://host:`, which is invalid.
- Evidence:
  - The same broken pattern appears in both the transport constructor and the sidebar helper that derives the favicon server origin.
- Why it matters:
  - Browser deployments using the default HTTPS or HTTP port can fail to connect at all.
  - Project favicon loading inherits the same malformed origin path.
- Fix direction:
  - Build the URL from `window.location.host`, or conditionally append `:${port}` only when `port` is non-empty.
  - Add tests for default-port browser environments.
- Suggested later cluster: `Cluster B: Transport and routing`

### BUG-005: In-flight WebSocket requests are lost on disconnect but do not reject until the 60s timeout expires

- Status: `Confirmed`
- Severity: `P2`
- Area: Web transport
- Affected files:
  - [apps/web/src/wsTransport.ts](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/wsTransport.ts#L75)
  - [apps/web/src/wsTransport.ts](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/wsTransport.ts#L177)
- Failure mode:
  - Requests are tracked in `pending` with a 60 second timeout.
  - When the socket closes, the transport clears `this.ws` and schedules a reconnect, but it does not reject or replay the in-flight pending requests.
- Evidence:
  - The close handler never iterates `pending`.
  - Only `dispose()` rejects pending requests immediately.
- Why it matters:
  - User actions can appear hung for a full minute after a disconnect, auth failure, or server restart even though the original request will never complete.
- Fix direction:
  - Reject pending requests on close with a transport-disconnected error, or implement explicit request replay semantics with idempotency/receipt handling.
- Suggested later cluster: `Cluster B: Transport and routing`

### BUG-006: Dev-mode HTTP redirects drop the requested path and query string

- Status: `Confirmed`
- Severity: `P2`
- Area: Dev server
- Affected files:
  - [apps/server/src/wsServer.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.ts#L491)
- Failure mode:
  - In dev mode, every HTTP request is redirected to `devUrl.href` exactly, regardless of the incoming pathname or search params.
- Evidence:
  - The handler returns `302` with `Location: devUrl.href`.
- Why it matters:
  - Refreshing a deep route or opening a copied link in dev loses the active thread, diff selection, or any route-level query state.
- Fix direction:
  - Preserve the incoming pathname and query string when rewriting to the Vite origin.
- Suggested later cluster: `Cluster B: Transport and routing`

### BUG-007: Workspace entry search cache is not invalidated after `projects.writeFile` writes

- Status: `Confirmed`
- Severity: `P2`
- Area: Server/web search
- Affected files:
  - [apps/server/src/workspaceEntries.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/workspaceEntries.ts#L12)
  - [apps/server/src/workspaceEntries.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/workspaceEntries.ts#L510)
  - [apps/server/src/workspaceEntries.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/workspaceEntries.ts#L537)
  - [apps/server/src/wsServer.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/wsServer.ts#L751)
- Failure mode:
  - Workspace search results are cached for 15 seconds.
  - The `projects.writeFile` route writes files but never clears that cache.
- Evidence:
  - `clearWorkspaceIndexCache()` exists, but the write route returns immediately after writing.
  - Cache invalidation is currently wired only in checkpoint/revert flows, not in manual workspace writes.
- Why it matters:
  - Newly created files can stay invisible to `@` mentions and workspace search immediately after they are written from the UI.
- Fix direction:
  - Clear the workspace index cache for `body.cwd` after successful writes and any other file-creation/deletion routes.
- Suggested later cluster: `Cluster B: Transport and routing`

### BUG-008: macOS hero download detection hardcodes Apple Silicon and misroutes Intel Macs

- Status: `Confirmed`
- Severity: `P3`
- Area: Marketing/download
- Affected files:
  - [apps/marketing/src/pages/index.astro](/Users/peterryszkiewicz/Repos/t3code/apps/marketing/src/pages/index.astro#L31)
- Failure mode:
  - Every Mac user agent is classified as `{ os: "mac", arch: "arm64" }`.
  - Asset selection then prefers the arm64 DMG.
- Evidence:
  - `detectPlatform()` never inspects `navigator.userAgentData`, CPU architecture hints, or Intel-vs-Apple-Silicon signals.
- Why it matters:
  - Intel Mac users are funneled to the wrong default artifact from the homepage hero button.
- Fix direction:
  - Detect architecture explicitly where possible and otherwise avoid arch-specific auto-selection from the hero CTA.
- Suggested later cluster: `Cluster C: UX and download flows`

### BUG-009: Newly created projects are appended to the bottom of the project list

- Status: `Confirmed`
- Severity: `P3`
- Area: Web UX
- Affected files:
  - [apps/web/src/store.ts](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/store.ts#L120)
  - [apps/web/src/store.ts](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/store.ts#L153)
  - [TODO.md](/Users/peterryszkiewicz/Repos/t3code/TODO.md#L8)
- Failure mode:
  - New projects get `orderIndex` after all existing projects whenever prior order exists, so they show up at the end.
- Evidence:
  - `orderIndex` falls back to `previous.length + incomingIndex` for new entries.
  - The repo TODO already tracks "New projects should go on top."
- Why it matters:
  - A newly created project is easy to miss because it appears below older projects instead of surfacing immediately.
- Fix direction:
  - Derive a default insertion policy for new projects that prefers recency, then layer manual drag order on top if the user has explicitly reordered.
- Suggested later cluster: `Cluster C: UX and download flows`

### BUG-010: Project ordering ignores latest thread activity and preserves stale manual/persisted order

- Status: `Confirmed`
- Severity: `P3`
- Area: Web UX
- Affected files:
  - [apps/web/src/store.ts](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/store.ts#L120)
  - [apps/web/src/store.ts](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/store.ts#L153)
  - [TODO.md](/Users/peterryszkiewicz/Repos/t3code/TODO.md#L9)
- Failure mode:
  - Project sync order is computed only from previous order or persisted order.
  - No thread activity, project `updatedAt`, or latest-turn signal is used to re-rank projects.
- Evidence:
  - `mapProjectsFromReadModel()` does not read thread state at all.
  - The repo TODO separately tracks "Projects should be sorted by latest thread update."
- Why it matters:
  - A project with fresh activity can stay buried below inactive projects, which slows navigation in active multi-project usage.
- Fix direction:
  - Split "manual pinned order" from "default recency order", or derive an activity-based order when the user has not explicitly rearranged projects.
- Suggested later cluster: `Cluster C: UX and download flows`

### BUG-011: Sidebar thread previews sort by creation time, so active older threads can stay buried

- Status: `High-confidence likely`
- Severity: `P3`
- Area: Web UX
- Affected files:
  - [apps/web/src/components/Sidebar.tsx](/Users/peterryszkiewicz/Repos/t3code/apps/web/src/components/Sidebar.tsx#L1296)
- Failure mode:
  - Thread lists are sorted by `createdAt` descending instead of any activity or update signal.
- Evidence:
  - The sidebar sorts project threads by `new Date(b.createdAt) - new Date(a.createdAt)`.
- Why it matters:
  - Once a project has many threads, an older but currently active thread can remain below a newer inactive thread.
- Fix direction:
  - Sort by latest relevant activity signal instead of raw creation time, or explicitly choose and document a different rule if this is intentional.
- Suggested later cluster: `Cluster C: UX and download flows`

### BUG-012: Desktop build emits `MODULE_TYPELESS_PACKAGE_JSON` warnings because the package is missing `"type": "module"`

- Status: `Confirmed`
- Severity: `P3`
- Area: Desktop tooling
- Affected files:
  - [apps/desktop/package.json](/Users/peterryszkiewicz/Repos/t3code/apps/desktop/package.json#L1)
- Failure mode:
  - The desktop package is consumed as ESM during build, but its local `package.json` does not declare `"type": "module"`.
- Evidence:
  - `bun run build` emitted Node `MODULE_TYPELESS_PACKAGE_JSON` warnings while processing the desktop build during this audit.
- Why it matters:
  - The build is noisier and more fragile than it needs to be, and future tooling upgrades can tighten this warning into a harder failure.
- Fix direction:
  - Add `"type": "module"` to the desktop package or make the desktop build inputs consistently CommonJS.
- Suggested later cluster: `Cluster D: Packaging and build health`

### BUG-013: Production web build emits oversized chunk warnings, including a roughly 2.5 MB minified main bundle

- Status: `Confirmed`
- Severity: `P2`
- Area: Web performance
- Affected files:
  - [apps/web/package.json](/Users/peterryszkiewicz/Repos/t3code/apps/web/package.json)
  - [apps/web/vite.config.ts](/Users/peterryszkiewicz/Repos/t3code/apps/web/vite.config.ts)
- Failure mode:
  - The production build currently generates chunks large enough for the build tool to warn, including an `index-*.js` bundle around 2.5 MB minified during this audit.
- Evidence:
  - `bun run build` emitted oversized chunk warnings and a plugin timing warning dominated by `@rolldown/plugin-babel`.
- Why it matters:
  - Performance is a stated project priority, and these bundle sizes increase startup cost and make later regressions harder to notice.
- Fix direction:
  - Split `ChatView`/diff/editor-heavy code paths more aggressively, audit large dependency pulls, and turn bundle-size checks into a tracked build budget.
- Suggested later cluster: `Cluster D: Packaging and build health`

## Verification Gaps And Repo-Tracked Issues

These are important, but this audit did not independently validate them enough to call them confirmed defects yet.

### GAP-001: Composer submit auto-scroll remains repo-tracked but was not reproduced in this audit

- Source:
  - [TODO.md](/Users/peterryszkiewicz/Repos/t3code/TODO.md#L5)
- Notes:
  - The repo still tracks "Submitting new messages should scroll to bottom."
  - The current scroll code is non-trivial and worth a targeted browser repro before scheduling a fix phase.

### GAP-002: Thread archiving is still tracked as missing UX/lifecycle work

- Source:
  - [TODO.md](/Users/peterryszkiewicz/Repos/t3code/TODO.md#L7)
- Notes:
  - This looks like a known lifecycle gap rather than a newly discovered regression.
  - It should be triaged during remediation planning, not silently folded into unrelated UX work.

### GAP-003: Message queueing remains an acknowledged missing reliability feature

- Source:
  - [TODO.md](/Users/peterryszkiewicz/Repos/t3code/TODO.md#L13)
- Notes:
  - The lack of queueing becomes more important once transport disconnect bugs are fixed, because the current system can still drop user intent during reconnect windows.

### GAP-004: Live-provider and packaged-update edge cases are still environment-gated

- Sources:
  - [apps/server/src/codexAppServerManager.test.ts](/Users/peterryszkiewicz/Repos/t3code/apps/server/src/codexAppServerManager.test.ts#L751)
  - [apps/desktop/src/main.ts](/Users/peterryszkiewicz/Repos/t3code/apps/desktop/src/main.ts)
- Notes:
  - Live Codex resume flows and packaged auto-update install paths were not exercised end-to-end in this audit session.
  - The catalog above stays confined to defects with enough local source or command evidence.

## Suggested Later Clusters

These are grouping hints only. The user explicitly asked to choose actual remediation phases later.

### Cluster A: Server exposure and path safety

- BUG-001
- BUG-002
- BUG-003

### Cluster B: Transport and routing

- BUG-004
- BUG-005
- BUG-006
- BUG-007

### Cluster C: UX and download flows

- BUG-008
- BUG-009
- BUG-010
- BUG-011

### Cluster D: Packaging and build health

- BUG-012
- BUG-013

---

_Initial bug catalog created on 2026-03-14._
_Use this file as the audit baseline before choosing remediation phases._
