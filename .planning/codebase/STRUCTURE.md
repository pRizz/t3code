# Codebase Structure

**Analysis Date:** 2026-03-14

## Directory Layout

```text
t3code/
├── .docs/                    # Reference documentation for maintainers
├── .github/workflows/        # CI, release, and repo automation
├── .plans/                   # Maintainability/refactor plans
├── apps/
│   ├── desktop/              # Electron shell
│   ├── marketing/            # Astro marketing site
│   ├── server/               # Node/Bun WebSocket + HTTP server
│   └── web/                  # React/Vite product UI
├── assets/                   # Shared packaged assets and icons
├── docs/                     # Release/user docs outside .docs
├── packages/
│   ├── contracts/            # Shared schema-only contracts
│   └── shared/               # Shared runtime utilities
├── scripts/                  # Repo-level build/release/dev scripts
├── AGENTS.md                 # Repo workflow instructions
├── package.json              # Workspace scripts and dependency catalog
├── tsconfig.base.json        # Shared TS compiler base
├── turbo.json                # Workspace task graph
└── vitest.config.ts          # Root test aliasing
```

## Directory Purposes

**`apps/server/`**

- Purpose: product backend and CLI package (`t3`)
- Key locations: `apps/server/src/index.ts`, `apps/server/src/main.ts`, `apps/server/src/wsServer.ts`, `apps/server/src/serverLayers.ts`
- Domain subfolders: `orchestration/`, `provider/`, `persistence/`, `git/`, `terminal/`, `checkpointing/`, `telemetry/`

**`apps/web/`**

- Purpose: React UI shared by browser and desktop
- Key locations: `apps/web/src/main.tsx`, `apps/web/src/router.ts`, `apps/web/src/components/`, `apps/web/src/routes/`, `apps/web/src/store.ts`

**`apps/desktop/`**

- Purpose: Electron wrapper around the shared server + web UI
- Key locations: `apps/desktop/src/main.ts`, `apps/desktop/src/preload.ts`

**`apps/marketing/`**

- Purpose: public landing/download pages
- Key locations: `apps/marketing/src/pages/index.astro`, `apps/marketing/src/lib/releases.ts`

**`packages/contracts/`**

- Purpose: app-agnostic schemas and protocol definitions only
- Key locations: `packages/contracts/src/ws.ts`, `packages/contracts/src/orchestration.ts`, `packages/contracts/src/providerRuntime.ts`

**`packages/shared/`**

- Purpose: runtime helpers with explicit subpath exports
- Key locations: `packages/shared/src/Net.ts`, `packages/shared/src/DrainableWorker.ts`, `packages/shared/src/schemaJson.ts`

**`scripts/`**

- Purpose: repo-level dev, release, and packaging workflows
- Key locations: `scripts/dev-runner.ts`, `scripts/build-desktop-artifact.ts`, `scripts/release-smoke.ts`

## Key File Locations

**Entry points:**

- `apps/server/src/index.ts` - server CLI bootstrap
- `apps/web/src/main.tsx` - web renderer bootstrap
- `apps/desktop/src/main.ts` - Electron main process
- `apps/marketing/src/pages/index.astro` - marketing homepage

**Protocol/contracts:**

- `packages/contracts/src/ws.ts`
- `packages/contracts/src/orchestration.ts`
- `packages/contracts/src/provider.ts`
- `packages/contracts/src/providerRuntime.ts`

**Runtime assembly:**

- `apps/server/src/serverLayers.ts`
- `apps/server/src/orchestration/Layers/*.ts`
- `apps/server/src/provider/Layers/*.ts`

**Client state and transport:**

- `apps/web/src/wsTransport.ts`
- `apps/web/src/wsNativeApi.ts`
- `apps/web/src/store.ts`
- `apps/web/src/components/ChatView.tsx`

**Testing and harnesses:**

- source-adjacent `*.test.ts` files across apps/packages
- `apps/server/integration/*.integration.test.ts`
- `apps/server/integration/OrchestrationEngineHarness.integration.ts`

## Naming Conventions

**Files:**

- Most TypeScript source files use `camelCase.ts` naming such as `wsTransport.ts`, `codexAppServerManager.ts`, and `projectScripts.ts`.
- React components often use `PascalCase.tsx`, for example `ChatView.tsx`, `BranchToolbar.tsx`, and `PlanSidebar.tsx`.
- TanStack Router files encode route shape in the filename, for example `apps/web/src/routes/_chat.$threadId.tsx`.
- Tests live beside code as `*.test.ts`; browser-focused tests use `*.browser.tsx`; server integration tests use `*.integration.test.ts`.

**Directories:**

- Server domain folders are noun-based and often contain `Layers/` plus `Services/`.
- Web folders separate by role: `components/`, `hooks/`, `lib/`, `routes/`.

**Generated/special files:**

- `apps/web/src/routeTree.gen.ts` is generated and ignored by lint/format configs.

## Where to Add New Code

**New protocol or model schema:**

- Add to `packages/contracts/src/*.ts`
- Re-export from `packages/contracts/src/index.ts` if it is part of the public package surface

**New shared runtime helper:**

- Add to `packages/shared/src/*.ts`
- Register an explicit subpath export in `packages/shared/package.json`

**New server domain behavior:**

- Prefer the existing domain folder under `apps/server/src/`
- Add service interfaces to `Services/` and implementations to `Layers/` when following the Effect pattern
- Wire cross-domain runtime changes through `apps/server/src/serverLayers.ts`

**New web feature:**

- Route files: `apps/web/src/routes/`
- View components: `apps/web/src/components/`
- Non-UI logic: `apps/web/src/lib/`, `apps/web/src/hooks/`, or standalone `*.logic.ts`

**New desktop-only capability:**

- `apps/desktop/src/`

**New repo automation or release logic:**

- `scripts/` and `.github/workflows/`

## Special Directories

**`.docs/`**

- Maintainer-facing architecture, CI, runtime, and script notes that complement the codebase map.

**`.plans/`**

- Existing refactor/maintainability plan history. Useful context before large changes.

**`apps/server/integration/`**

- Integration harnesses and higher-fidelity server/provider tests.

**`assets/`**

- Shared binary/package assets used during desktop packaging.

**`.planning/codebase/`**

- Target location for this codebase map. Future refreshes should update these docs rather than scattering repo-understanding notes elsewhere.

---

_Structure analysis: 2026-03-14_
_Update when directory layout, ownership boundaries, or placement rules change_
