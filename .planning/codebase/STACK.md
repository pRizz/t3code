# Technology Stack

**Analysis Date:** 2026-03-14

## Languages

- TypeScript is the primary implementation language across `apps/server`, `apps/web`, `apps/desktop`, `packages/contracts`, `packages/shared`, and `scripts`.
- TSX is used for the React renderer in `apps/web/src/*.tsx`.
- Astro component files are used in `apps/marketing/src/pages/*.astro`.
- JSON, TOML, YAML, and Markdown drive repo configuration and internal documentation in files such as `package.json`, `.mise.toml`, `.github/workflows/*.yml`, and `.docs/*.md`.

## Runtime

- Bun is the package manager and part of the development/runtime story. Root `package.json` pins `bun@1.3.9`, and `.mise.toml` pins Bun for contributors.
- Node.js is still a first-class runtime requirement. Root `package.json` and `.mise.toml` pin Node `24.13.1`, the server bundles for Node, and CI installs both Node and Bun.
- The web UI runs in the browser through Vite during development and as static assets in production from `apps/web/dist`.
- The desktop app runs on Electron 40 and starts the shared server backend as a child process from `apps/desktop/src/main.ts`.
- The server persists state locally with SQLite through `@effect/sql-sqlite-bun` and expects a locally installed/authenticated Codex CLI for the core provider workflow.

## Frameworks

- Effect is the dominant backend/runtime framework. The server uses `Effect`, `Layer`, `ServiceMap`, `Schema`, and `@effect/platform-node` throughout `apps/server/src/*` and `packages/contracts/src/*`.
- React 19 powers the main product UI in `apps/web`, with TanStack Router for file-based routing and TanStack Query for server interaction caching.
- Tailwind CSS v4 and `@base-ui/react` provide the styling and primitive UI layer for the web app.
- Electron powers the desktop shell in `apps/desktop`, while the product server remains the same `t3` backend used by the web mode.
- Astro 6 powers the marketing site in `apps/marketing`.
- Turbo coordinates workspace builds and checks from the monorepo root via `turbo.json`.

## Key Dependencies

- `effect`, `@effect/platform-node`, and `@effect/sql-sqlite-bun` underpin server services, schemas, persistence, and runtime composition.
- `ws` provides the WebSocket transport between browser/desktop clients and `apps/server/src/wsServer.ts`.
- `node-pty` and the terminal layers in `apps/server/src/terminal/*` provide interactive terminal sessions for threads.
- `@tanstack/react-router`, `@tanstack/react-query`, `zustand`, `@xterm/xterm`, and `@xterm/addon-fit` drive the main web application UX.
- `electron` and `electron-updater` power the desktop distribution and auto-update flow.
- `vitest`, `@effect/vitest`, `oxlint`, `oxfmt`, and `tsdown` are the main quality/build tools.

## Configuration

- Root orchestration lives in `package.json`, `turbo.json`, and `tsconfig.base.json`.
- Formatting and linting are configured with `.oxfmtrc.json` and `.oxlintrc.json`.
- App/package-specific TypeScript and bundling configs live beside the package code, for example `apps/web/vite.config.ts`, `apps/server/tsdown.config.ts`, and `apps/desktop/tsdown.config.ts`.
- Environment-driven runtime behavior is centralized around `T3CODE_*`, `VITE_*`, `PORT`, and desktop/release variables declared in `turbo.json`, `apps/server/src/main.ts`, `scripts/dev-runner.ts`, and `.github/workflows/release.yml`.

## Platform Requirements

- Local development expects both Bun and Node installed at the pinned versions from `.mise.toml`.
- Core product usage requires Codex CLI to be installed and authorized, as noted in `README.md` and enforced by the provider stack in `apps/server/src/codexAppServerManager.ts`.
- Desktop packaging targets macOS (`dmg` arm64 and x64), Linux (`AppImage`), and Windows (`nsis`) through root release scripts and `.github/workflows/release.yml`.
- Browser tests require Playwright/Chromium via `apps/web/vitest.browser.config.ts`.
- Optional git/GitHub workflows assume local Git and `gh` CLI availability through `apps/server/src/git/Services/GitHubCli.ts`.

---

_Stack analysis: 2026-03-14_
_Update after major dependency, runtime, or build-pipeline changes_
