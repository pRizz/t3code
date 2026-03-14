# Testing Patterns

**Analysis Date:** 2026-03-14

## Test Framework

**Runner:**

- Vitest is the primary test runner across the monorepo.
- Root config: `vitest.config.ts`
- App-specific configs: `apps/server/vitest.config.ts`, `apps/web/vitest.browser.config.ts`

**Assertion libraries:**

- `expect` from Vitest is common in web/unit tests.
- `assert` and `it.effect`/`it.layer` from `@effect/vitest` are common in server, contracts, and shared package tests.

**Run commands:**

```bash
bun run test
bun run --cwd apps/web test:browser
bun run test:desktop-smoke
```

## Test File Organization

- Most tests live beside the code they exercise as `*.test.ts`.
- Browser-only tests live in `apps/web/src/*.browser.tsx` and are explicitly included by `apps/web/vitest.browser.config.ts`.
- Higher-fidelity server tests live in `apps/server/integration/*.integration.test.ts`.
- Support harnesses and fixtures live beside integration tests, for example:
  - `apps/server/integration/OrchestrationEngineHarness.integration.ts`
  - `apps/server/integration/TestProviderAdapter.integration.ts`
  - `apps/server/integration/fixtures/providerRuntime.ts`

## Test Structure

- Web and utility tests typically use `describe(...)` plus `it(...)`.
- Effect-heavy modules often use `it.effect(...)`, `it.layer(...)`, or `it.live(...)` with `Effect.gen(...)`.
- Resourceful tests frequently use `Effect.acquireUseRelease(...)` or helper harnesses to make startup/shutdown deterministic.
- The repo instructions ask for Arrange/Act/Assert-style thinking, but the codebase does not use mandatory AAA comments everywhere. Structure is usually expressed through helper setup plus focused assertions instead.

## Mocking

- `vi.fn()` is the default mocking tool.
- The codebase often prefers lightweight in-memory test doubles over deep mocking:
  - `MockWebSocket` in `apps/web/src/wsTransport.test.ts`
  - `MockTerminalManager` in `apps/server/src/wsServer.test.ts`
- Effect service substitution via `Layer.succeed(...)` is a common server-side pattern for isolating dependencies.
- Browser/global environment mocking is done by overriding `window`, `WebSocket`, or desktop bridge shims in setup blocks.

## Fixtures and Factories

- Many tests use local factory helpers rather than a single shared fixtures package.
- Common patterns include branded ID helpers (`asThreadId`, `asEventId`) and object builders like `makeActivity(...)`.
- Integration tests lean on reusable harnesses instead of snapshots or large fixture directories.
- There is no broad repo-wide `tests/factories/` convention at the moment.

## Coverage

- No explicit coverage threshold or coverage-report command was found in root scripts or CI.
- CI focuses on passing format/lint/typecheck/tests/browser tests/build rather than numeric coverage gating.
- Browser coverage is narrow because `apps/web/vitest.browser.config.ts` currently includes only two browser-rendered component tests.

## Test Types

**Unit tests:**

- Most `*.test.ts` files across `apps/web`, `packages/contracts`, and `packages/shared`
- Focus on pure helpers, schema decoding, store logic, transport behavior, and small service units

**Integration tests:**

- Primarily in `apps/server/integration/`
- Use real SQLite/git/runtime wiring with test harnesses
- Some tests exercise end-to-end orchestration flow from command dispatch through checkpoints and read models

**Browser/component tests:**

- `apps/web/src/components/ChatView.browser.tsx`
- `apps/web/src/components/KeybindingsToast.browser.tsx`

**Smoke tests:**

- Desktop smoke flow via `apps/desktop/scripts/smoke-test.mjs`
- Release smoke via `scripts/release-smoke.ts`

## Common Patterns

**Effect-based test setup:**

```typescript
it.layer(testLayer)("example", (it) => {
  it.effect("does something", () =>
    Effect.gen(function* () {
      // setup, execute, assert
    }),
  );
});
```

**Transport tests with manual channels/doubles:**

- Mock socket classes or message queues are favored over spinning up full external infrastructure when the transport logic itself is the subject under test.

**Environment-gated real-provider coverage:**

- Some Codex-backed integration tests are intentionally skipped unless a real provider binary path is present, for example `skipIf(!process.env.CODEX_BINARY_PATH)` in `apps/server/integration/orchestrationEngine.integration.test.ts`.

---

_Testing analysis: 2026-03-14_
_Update when test runners, coverage gates, or integration strategy change_
