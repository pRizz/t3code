# Coding Conventions

**Analysis Date:** 2026-03-14

## Naming Patterns

**Files:**

- `camelCase` is the dominant naming style for non-component TypeScript files, for example `wsTransport.ts`, `session-logic.ts` is a notable exception, and `codexAppServerManager.ts`.
- `PascalCase` is common for React components and server Effect layer/service files such as `ChatView.tsx`, `ProviderService.ts`, and `OrchestrationEngine.ts`.
- Tests stay beside source files with a `.test.ts` suffix. Browser tests use `.browser.tsx`, and server integration tests use `.integration.test.ts`.

**Functions and helpers:**

- `camelCase` for functions, variables, and methods
- Repeated prefixes reflect intent:
  - `build*` for constructors/derived values
  - `resolve*` for lookup/defaulting logic
  - `derive*` for UI projections
  - `map*` and `to*` for transformations
  - `handle*` for event handlers

**Types and services:**

- `PascalCase` for schemas, branded IDs, types, and service tags
- `*Shape` names define service interfaces
- `*Live` names define Effect layer implementations
- Tagged errors are named with `*Error`

## Code Style

- Formatting is enforced with `oxfmt` from the root `fmt` script.
- The repo uses double quotes, semicolons, trailing commas, and relatively tight line wrapping, matching the source in `apps/*` and `packages/*`.
- Strict TypeScript is enabled in `tsconfig.base.json`, including `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, and `noImplicitOverride`.
- Early returns and guard clauses are common in both UI and server code.
- Server modules heavily favor `Effect.gen(function* () {})` and pipeline-based composition instead of ad hoc promise chains.

## Import Organization

- Imports are usually grouped as:
  1. Node built-ins
  2. external packages
  3. workspace packages such as `@t3tools/contracts` and `@t3tools/shared/*`
  4. local relative imports
- Blank lines between import groups are common and consistent.
- The web app uses `~/` as a local alias from `apps/web/tsconfig.json`.
- `packages/shared` intentionally uses explicit subpath exports and no root barrel export. That constraint is documented in repo instructions and enforced by `packages/shared/package.json`.

## Error Handling

- Server-side code prefers typed errors at boundaries using `Schema.TaggedErrorClass`, `Data.TaggedError`, and explicit error mappers such as `toPersistenceSqlError(...)`.
- Schema decoding is used to reject malformed inputs at the protocol edge rather than deep in the implementation.
- Browser/UI code often catches and swallows non-critical failures to avoid breaking the chat surface, especially in storage listeners, WebSocket listeners, and fallback UI helpers.
- Tests sometimes use `as unknown as` for mocks and harnesses, but production code is generally more disciplined than the test harness surface.

## Logging

- Server logging uses `Effect.log*` and a local logger factory in `apps/server/src/logger.ts`.
- Browser transport code logs malformed inbound frames with `console.warn` in `apps/web/src/wsTransport.ts`.
- Desktop packaged builds use rotating log files via `RotatingFileSink` in `apps/desktop/src/main.ts`.
- Telemetry is centralized on the server through `apps/server/src/telemetry/Layers/AnalyticsService.ts`.

## Comments

- Comments are relatively sparse.
- When present, they are usually there to explain a boundary condition, parser quirk, sequencing requirement, or operational reason rather than narrating obvious code.
- Good examples live around WebSocket readiness, route generation quirks, packaging behavior, and React/Vite compiler edge cases.

## Function Design

- Small pure helpers are common around larger workflows, especially `to*`, `read*`, and `normalize*` helpers.
- Options objects are preferred once a function has several related parameters, especially on the server side.
- UI logic is often extracted into `*.logic.ts` or helper modules when it can be tested independently of React rendering.
- The codebase does contain some very large coordinator modules, but those are exceptions already tracked in `.plans/`.

## Module Design

- Server domains follow a repeatable structure:
  - `Services/` for interfaces/tags
  - `Layers/` for concrete implementations
  - top-level domain files for shared errors/helpers/schemas
- Contracts stay schema-only in `packages/contracts`; runtime behavior belongs elsewhere.
- Web modules frequently split pure logic from UI shells, for example `BranchToolbar.logic.ts` beside `BranchToolbar.tsx`.
- Default exports are mostly used for route/component entry modules, while named exports are the norm for shared logic.

---

_Convention analysis: 2026-03-14_
_Update when formatting, naming, or module-shape norms materially change_
