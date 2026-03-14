# External Integrations

**Analysis Date:** 2026-03-14

## APIs & External Services

**Codex runtime:**

- `codex app-server` is the primary external dependency for product behavior.
  - Integration path: `apps/server/src/codexAppServerManager.ts`
  - Protocol: JSON-RPC over stdio
  - Role: session startup/resume, turn execution, approvals, structured user input, model metadata

**GitHub APIs and tooling:**

- GitHub CLI (`gh`) is used for pull request and repository workflows.
  - Integration path: `apps/server/src/git/Layers/GitHubCli.ts`
  - Role: branch/PR lookup, checkout, default branch resolution, PR creation
- GitHub Releases REST API is queried by the marketing site.
  - Integration path: `apps/marketing/src/lib/releases.ts`
  - Endpoint: `https://api.github.com/repos/pingdotgg/t3code/releases/latest`
  - Role: platform-aware download button selection

**Release/update infrastructure:**

- Desktop updates are driven by `electron-updater` and GitHub Releases metadata.
  - Integration path: `apps/desktop/src/main.ts`
  - Role: check/download/install desktop updates

**System integrations:**

- The server and desktop app open local editors, browsers, and external URLs.
  - Server path: `apps/server/src/open.ts`
  - Desktop path: `apps/desktop/src/main.ts`
  - Role: open project folders, external docs, and system-native dialogs

## Data Storage

**Primary app state:**

- SQLite is the main persistent store for orchestration state.
  - Integration path: `apps/server/src/persistence/*`
  - Client: `@effect/sql-sqlite-bun`
  - Data stored: event store, command receipts, projections, provider session runtime

**Git repositories and checkpoints:**

- User workspaces are treated as git repos for diffing, checkpoint capture, and worktree flows.
  - Integration path: `apps/server/src/checkpointing/*`, `apps/server/src/git/*`
  - Storage model: git refs + diff blobs stored locally

**Local files and attachments:**

- State directory storage holds logs, keybindings, telemetry identity, and provider artifacts.
  - Config entry point: `apps/server/src/main.ts`, `apps/server/src/config.ts`
- Attachment files are stored locally and served back via `apps/server/src/attachmentStore.ts` and `apps/server/src/attachmentPaths.ts`.

**Client-side storage:**

- The web app uses `localStorage` in `apps/web/src/store.ts` and related modules for UI preferences/state.
- The marketing site caches release metadata in `sessionStorage` via `apps/marketing/src/lib/releases.ts`.

## Authentication & Identity

**Provider authentication:**

- Codex auth is delegated to the local Codex installation.
  - Integration path: `apps/server/src/codexAppServerManager.ts`
  - Modes observed: ChatGPT account state or API key-backed Codex auth snapshot

**App transport authentication:**

- WebSocket/auth protection is optional in standalone server mode and explicit in desktop mode.
  - Config path: `apps/server/src/main.ts`
  - Desktop wiring: generated auth token passed from `apps/desktop/src/main.ts`

**Application user accounts:**

- There is no first-party multi-user auth system in this repo. The product is a local-first single-user tool rather than a hosted SaaS backend.

## Monitoring & Observability

**Product telemetry:**

- Anonymous PostHog telemetry is sent from the server.
  - Integration path: `apps/server/src/telemetry/Layers/AnalyticsService.ts`
  - Config: `T3CODE_POSTHOG_KEY`, `T3CODE_POSTHOG_HOST`, `T3CODE_TELEMETRY_ENABLED`

**Runtime/event logs:**

- Provider native and canonical event streams are written as NDJSON logs under the server state dir.
  - Integration path: `apps/server/src/serverLayers.ts`, `apps/server/src/provider/Layers/EventNdjsonLogger.ts`
- Desktop packaged builds write rotating logs via `RotatingFileSink`.
  - Integration path: `apps/desktop/src/main.ts`

## CI/CD & Deployment

**CI:**

- GitHub Actions runs formatting, lint, typecheck, tests, browser tests, and desktop build verification.
  - Workflow: `.github/workflows/ci.yml`

**Release publishing:**

- GitHub Actions builds signed/unsigned desktop artifacts and publishes npm + GitHub release assets.
  - Workflow: `.github/workflows/release.yml`

**Desktop signing:**

- macOS uses Apple signing/notarization secrets.
- Windows uses Azure Trusted Signing environment variables.

## Environment Configuration

**Core runtime variables:**

- `T3CODE_MODE`, `T3CODE_PORT`, `T3CODE_HOST`, `T3CODE_STATE_DIR`, `T3CODE_AUTH_TOKEN`, `T3CODE_NO_BROWSER`, `T3CODE_AUTO_BOOTSTRAP_PROJECT_FROM_CWD`, and `T3CODE_LOG_WS_EVENTS`

**Dev workflow variables:**

- `PORT`, `VITE_WS_URL`, `VITE_DEV_SERVER_URL`, `ELECTRON_RENDERER_PORT`, `T3CODE_DEV_INSTANCE`, and `T3CODE_PORT_OFFSET`

**Telemetry/release variables:**

- `T3CODE_POSTHOG_*` for telemetry
- Apple and Azure signing variables from `.github/workflows/release.yml`

**Where they are defined/consumed:**

- `turbo.json`
- `apps/server/src/main.ts`
- `scripts/dev-runner.ts`
- `.github/workflows/*.yml`

## Webhooks & Callbacks

**Incoming webhooks:**

- None in the application runtime. The product is not exposing SaaS webhook handlers.

**Outgoing callbacks/background polling:**

- Desktop auto-update checks GitHub release metadata through `electron-updater`.
- Marketing download UX fetches GitHub Releases metadata from the browser.
- GitHub Actions reacts to push, pull request, issue comment, and tag events in `.github/workflows/*`.

---

_Integration audit: 2026-03-14_
_Update when adding or removing external systems, secrets, or deployment flows_
