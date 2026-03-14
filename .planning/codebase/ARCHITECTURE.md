# Architecture

**Analysis Date:** 2026-03-14

## Pattern Overview

- This repo is a workspace monorepo with one shared product architecture and two user-facing shells:
  - a local Node/Bun-backed WebSocket server in `apps/server`
  - a shared React UI in `apps/web`
  - an Electron wrapper in `apps/desktop`
  - a separate Astro marketing site in `apps/marketing`
- The core product pattern is server-authoritative and event-sourced. Provider-native runtime events are normalized into orchestration domain events, persisted to SQLite, projected into a read model, and pushed to clients over typed WebSocket channels.
- Shared contracts in `packages/contracts/src/*.ts` define the protocol boundary. Shared runtime helpers in `packages/shared/src/*.ts` hold reusable implementation utilities without becoming a general barrel package.

## Layers

**Contracts layer:**

- Purpose: schema-only protocol/model definitions
- Location: `packages/contracts/src/*.ts`
- Key files: `packages/contracts/src/ws.ts`, `packages/contracts/src/orchestration.ts`, `packages/contracts/src/providerRuntime.ts`

**Shared runtime layer:**

- Purpose: reusable logic shared by apps without pulling in app-specific behavior
- Location: `packages/shared/src/*.ts`
- Key files: `packages/shared/src/Net.ts`, `packages/shared/src/DrainableWorker.ts`, `packages/shared/src/logging.ts`

**Server runtime layer:**

- Purpose: own process config, provider orchestration, persistence, terminals, git, telemetry, and WebSocket routing
- Location: `apps/server/src/**/*`
- Assembly point: `apps/server/src/serverLayers.ts`

**Client application layer:**

- Purpose: React routes, UI state, typed transport client, and conversation UX
- Location: `apps/web/src/**/*`
- Key files: `apps/web/src/main.tsx`, `apps/web/src/wsTransport.ts`, `apps/web/src/wsNativeApi.ts`, `apps/web/src/store.ts`

**Desktop shell layer:**

- Purpose: package the server + web UI into a desktop app and bridge Electron-only capabilities
- Location: `apps/desktop/src/*`
- Key file: `apps/desktop/src/main.ts`

## Data Flow

**Startup flow:**

1. `apps/server/src/index.ts` builds the Effect runtime and CLI command entry.
2. `apps/server/src/main.ts` resolves environment/CLI configuration and composes the server/provider layers.
3. `apps/server/src/wsServer.ts` starts HTTP + WebSocket listeners, waits on `apps/server/src/wsServer/readiness.ts`, and then emits ordered welcome/config pushes through `apps/server/src/wsServer/pushBus.ts`.
4. `apps/web/src/main.tsx` boots the React app, while `apps/web/src/wsNativeApi.ts` and `apps/web/src/wsTransport.ts` subscribe to typed push channels and expose a `NativeApi` shape to the UI.

**User turn flow:**

1. UI actions call the `NativeApi` in `apps/web/src/wsNativeApi.ts`.
2. `apps/server/src/wsServer.ts` decodes the request with schemas from `packages/contracts/src/ws.ts`.
3. Intent events are routed into the orchestration/provider stack built in `apps/server/src/serverLayers.ts`.
4. `apps/server/src/orchestration/Layers/ProviderCommandReactor.ts` starts provider work through `apps/server/src/provider/Layers/ProviderService.ts`.
5. `apps/server/src/codexAppServerManager.ts` talks to `codex app-server` over stdio JSON-RPC.
6. Provider-native events flow into `apps/server/src/orchestration/Layers/ProviderRuntimeIngestion.ts`, which dispatches canonical orchestration events into `apps/server/src/orchestration/Layers/OrchestrationEngine.ts`.
7. Projection/persistence layers update SQLite-backed read models and `apps/server/src/wsServer.ts` pushes resulting domain events to the client.

**Desktop flow:**

1. `apps/desktop/src/main.ts` starts the packaged server backend as a child process.
2. It generates a per-run auth token and WebSocket URL for the BrowserWindow preload bridge.
3. The BrowserWindow loads either the Vite dev URL or the packaged web bundle while reusing the same web app transport model.

## Key Abstractions

**Effect services and layers:**

- Most server domains expose a `*Shape` interface, a `ServiceMap.Service` tag in `Services/`, and a `*Live` implementation in `Layers/`.
- Examples: `ProviderService`, `OrchestrationEngineService`, `GitManager`, `TerminalManager`

**Schema-first boundaries:**

- All protocol and model boundaries use Effect Schema definitions in `packages/contracts`.
- The WebSocket boundary is validated at both ends by `packages/contracts/src/ws.ts`, `apps/server/src/wsServer.ts`, and `apps/web/src/wsTransport.ts`.

**Queue-backed background workers:**

- Long-running ingestion/reaction pipelines use `makeDrainableWorker` from `packages/shared/src/DrainableWorker.ts`.
- Examples: `ProviderRuntimeIngestion`, `ProviderCommandReactor`, `CheckpointReactor`

**State projection:**

- The server owns canonical state in the orchestration read model and SQLite projections.
- The client mirrors that state into UI-friendly structures in `apps/web/src/store.ts`.

## Entry Points

**Server CLI/runtime entry:**

- `apps/server/src/index.ts`
- `apps/server/src/main.ts`

**Web app entry:**

- `apps/web/src/main.tsx`
- `apps/web/src/router.ts`

**Desktop app entry:**

- `apps/desktop/src/main.ts`
- `apps/desktop/src/preload.ts`

**Dev/build orchestration:**

- `scripts/dev-runner.ts`
- root `package.json`

**Marketing entry:**

- `apps/marketing/src/pages/index.astro`

## Error Handling

- Server code favors typed failures with `Schema.TaggedErrorClass` or `Data.TaggedError`, plus `Effect.catch`, `Effect.catchTag`, and explicit error translation helpers.
- Schema decoding happens at protocol boundaries so invalid input is rejected before domain logic runs.
- Browser code is more defensive than strict: transport listener/storage failures are frequently swallowed or downgraded to `console.warn` to preserve UX.
- Desktop startup/update flows catch and log operational failures instead of crashing the whole app when a non-critical capability fails.

## Cross-Cutting Concerns

- Ordered push delivery matters. `apps/server/src/wsServer/pushBus.ts` centralizes monotonic push sequencing.
- Persistence is local-first. SQLite, git refs, attachment files, and logs all live under local state/workspace directories rather than remote infrastructure.
- Telemetry and logging are server/desktop responsibilities, not client-side analytics-heavy web architecture.
- Git is a first-class runtime concern because thread diffs, checkpoints, worktrees, and PR helpers are built into the product.
- The repo is already carrying active maintainability work in `.plans/*.md`, especially around splitting large modules and hardening provider/runtime flows.

---

_Architecture analysis: 2026-03-14_
_Update when core patterns, layering, or runtime boundaries materially change_
