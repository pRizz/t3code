---
status: testing
phase: 09-implement-the-bug-001-fix-and-produce-an-upstream-pr-report
source:
  - 09-01-SUMMARY.md
  - 09-02-SUMMARY.md
started: 2026-03-14T10:58:33Z
updated: 2026-03-14T10:58:33Z
---

## Current Test

number: 1
name: Upstream PR Report Reads as a Standalone Reviewer Brief
expected: |
Open `.planning/phases/09-implement-the-bug-001-fix-and-produce-an-upstream-pr-report/09-UPSTREAM-PR-REPORT.md`.

You should be able to understand BUG-001 without rereading the full audit catalog first.
The report should clearly contain:

- `What Changed`
- `Why`
- `Steps to reproduce`
- `Fix summary`
- `Validation`

The reproduction path should read like a maintainer-friendly workflow, not loose notes, and it should clearly explain both the vulnerable behavior and the value of the fix.
awaiting: user response

## Tests

### 1. Upstream PR Report Reads as a Standalone Reviewer Brief

expected: Open the BUG-001 PR report and confirm it is self-contained, reviewer-friendly, and clearly explains the vulnerable behavior, the fix, and the validation evidence.
result: pending

### 2. In-Workspace Write Still Succeeds Through `projects.writeFile`

expected: |
Start the server from the repo root and leave it running:

`bun run --cwd apps/server dev -- --port 4321 --host 127.0.0.1 --no-browser`

In another terminal, run:

```sh
WORKSPACE=$(mktemp -d)
export WORKSPACE

node --input-type=module <<'EOF'
const workspace = process.env.WORKSPACE;
const ws = new WebSocket('ws://127.0.0.1:4321/');
const requestId = crypto.randomUUID();

ws.addEventListener('message', (event) => {
  const message = JSON.parse(String(event.data));
  if (message.type === 'push' && message.channel === 'server.welcome') {
    ws.send(JSON.stringify({
      id: requestId,
      body: {
        _tag: 'projects.writeFile',
        cwd: workspace,
        relativePath: 'plans/uat-happy.txt',
        contents: 'phase-9-happy-path\n'
      }
    }));
    return;
  }

  if (message.id === requestId) {
    console.log(JSON.stringify(message, null, 2));
    ws.close();
  }
});
EOF

cat "$WORKSPACE/plans/uat-happy.txt"
```

Expected result: the WebSocket response contains `result.relativePath: "plans/uat-happy.txt"` and the file exists inside the workspace with the exact contents `phase-9-happy-path`.
result: pending

### 3. Symlink Escape Attempt Is Rejected Without Outside Mutation

expected: |
Reuse the running server from Test 2.

In another terminal, run:

```sh
WORKSPACE=$(mktemp -d)
OUTSIDE=$(mktemp -d)
export WORKSPACE OUTSIDE

ln -s "$OUTSIDE" "$WORKSPACE/linked-outside"

node --input-type=module <<'EOF'
import { existsSync } from 'node:fs';

const workspace = process.env.WORKSPACE;
const outside = process.env.OUTSIDE;
const ws = new WebSocket('ws://127.0.0.1:4321/');
const requestId = crypto.randomUUID();

ws.addEventListener('message', (event) => {
  const message = JSON.parse(String(event.data));
  if (message.type === 'push' && message.channel === 'server.welcome') {
    ws.send(JSON.stringify({
      id: requestId,
      body: {
        _tag: 'projects.writeFile',
        cwd: workspace,
        relativePath: 'linked-outside/escape.txt',
        contents: 'owned\n'
      }
    }));
    return;
  }

  if (message.id === requestId) {
    console.log(JSON.stringify(message, null, 2));
    console.log('outside-exists', existsSync(`${outside}/escape.txt`));
    ws.close();
  }
});
EOF
```

Expected result: the response contains an error message with `Workspace file path must stay within the project root.` and the printed `outside-exists` value is `false`.
result: pending

## Summary

total: 3
passed: 0
issues: 0
pending: 3
skipped: 0

## Gaps

None yet.
