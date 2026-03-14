# Codebase Concerns

**Analysis Date:** 2026-03-14

## Tech Debt

**Oversized coordinator files remain in critical paths:**

- Files:
  - `apps/web/src/components/ChatView.tsx` (3930 lines)
  - `apps/server/src/codexAppServerManager.ts` (1587 lines)
  - `apps/server/src/git/Layers/GitCore.ts` (1428 lines)
  - `apps/desktop/src/main.ts` (1391 lines)
  - `apps/server/src/wsServer.ts` (1006 lines)
- Why: feature growth has outpaced modular extraction in the main coordinator modules.
- Impact: review difficulty, higher regression risk, and expensive mental context switching.
- Existing evidence: `.plans/03-split-codex-app-server-manager.md` and `.plans/04-split-chatview-component.md` already track this direction.

## Known Bugs

**Marketing site assumes Apple Silicon by default:**

- File: `apps/marketing/src/pages/index.astro`
- Symptom: `detectPlatform()` returns `{ os: "mac", arch: "arm64" }` for any Mac user agent.
- Impact: Intel Mac users may be pointed at the wrong default DMG before falling back manually.

**Several UX issues are still tracked but unresolved:**

- Tracking file: `TODO.md`
- Examples: submit should scroll to bottom, thread archiving, project sorting by latest thread update, queueing messages
- Impact: user-facing rough edges remain in core chat/project navigation flows.

## Security Considerations

**Server exposure depends on operator configuration:**

- Files: `apps/server/src/main.ts`, `apps/server/src/config.ts`, `apps/server/src/wsServer.ts`
- Risk: web mode can run without an auth token and may bind on a non-loopback host if the process is started that way.
- Current mitigation: desktop mode forces loopback + auth token, and workspace write routes validate relative paths.
- Recommendation: default standalone server mode to loopback-only or require auth when binding beyond localhost.

**Operational logs can contain sensitive local context:**

- Files: `apps/server/src/serverLayers.ts`, `apps/server/src/provider/Layers/EventNdjsonLogger.ts`, `apps/desktop/src/main.ts`
- Risk: prompts, file paths, provider events, and runtime details are written to local log files under the state dir.
- Recommendation: treat state/log directories as sensitive and document/expand opt-out or redaction behavior.

## Performance Bottlenecks

**Runtime cost grows with active sessions:**

- Files: `apps/server/src/provider/Layers/ProviderService.ts`, `apps/server/src/codexAppServerManager.ts`, `apps/server/src/terminal/Layers/Manager.ts`
- Problem: each active thread/session can imply a provider child process, terminal resources, event queues, and projection work.
- Impact: local resource usage will rise roughly linearly with concurrent active sessions.

**Client projection work is centralized in one large store sync path:**

- File: `apps/web/src/store.ts`
- Problem: the app rebuilds projected thread/project UI state from orchestration snapshots in one place.
- Impact: probably fine at current scale, but large thread histories or many projects could make UI state sync more expensive.

## Fragile Areas

**Provider-runtime ingestion pipeline is sequencing-sensitive:**

- Files:
  - `apps/server/src/provider/Layers/ProviderService.ts`
  - `apps/server/src/orchestration/Layers/ProviderRuntimeIngestion.ts`
  - `apps/server/src/orchestration/Layers/ProviderCommandReactor.ts`
  - `apps/server/src/orchestration/Layers/CheckpointReactor.ts`
- Why fragile: turn lifecycle, approvals, checkpoints, and receipts all depend on ordered async event handling.
- Safe modification path: validate with integration tests, not just unit tests.

**`wsServer.ts` mixes many responsibilities:**

- File: `apps/server/src/wsServer.ts`
- Why fragile: HTTP routes, WebSocket routing, startup bootstrap, auth, attachments, and push fan-out all converge here.
- Impact: small route or startup changes can accidentally affect unrelated server behavior.

## Scaling Limits

**Single-process local architecture:**

- Files: `apps/server/src/orchestration/Layers/OrchestrationEngine.ts`, `apps/server/src/persistence/Layers/*.ts`, `apps/server/src/wsServer/pushBus.ts`
- Limit: current design assumes one local server process with in-memory queues/pubsubs and a local SQLite store.
- Impact: strong fit for local tooling, poor fit for horizontal multi-instance deployment without re-architecture.

**Provider surface is broader in contracts than in implementation:**

- Files: `packages/contracts/src/*`, `apps/server/src/provider/Layers/CodexAdapter.ts`
- Limit: contracts and UI abstractions leave room for more providers, but the runtime is still effectively Codex-only today.

## Dependencies at Risk

**Effect catalog packages come from `pkg.pr.new`:**

- File: `package.json`
- Risk: prerelease URL-pinned packages can be less reproducible and may disappear/change more abruptly than registry-published stable versions.

**Web toolchain is intentionally near the edge:**

- Files: `apps/web/package.json`, `apps/web/vite.config.ts`
- Risk: React compiler beta, Vite 8, and Rolldown/Babel integration give good performance direction but likely increase upgrade/debug friction.

## Missing Critical Features

**Message queueing and richer thread lifecycle management are still pending:**

- Tracking file: `TODO.md`
- Gaps: queued sends, thread archiving, better ordering, and related session UX polish

**Claude Code support is not implemented yet:**

- Files: `README.md`, repo instructions, current provider code under `apps/server/src/provider/*`
- Gap: product messaging references future Claude Code support, but the implemented provider path remains Codex-first only.

## Test Coverage Gaps

**Browser coverage is intentionally narrow:**

- File: `apps/web/vitest.browser.config.ts`
- Gap: only two browser-rendered test files are included today, leaving most interactive chat flows covered by unit tests or manual testing.

**Real Codex integration coverage is environment-gated:**

- File: `apps/server/integration/orchestrationEngine.integration.test.ts`
- Gap: important live-provider tests are skipped unless the environment exposes a real Codex binary path, so CI mostly exercises the test adapter path.

**Coverage thresholds are not enforced:**

- Files: root `package.json`, `.github/workflows/ci.yml`
- Gap: CI ensures tests run, but not that a minimum coverage floor is maintained.

---

_Concerns audit: 2026-03-14_
_Update as issues are fixed, split out, or newly discovered_
