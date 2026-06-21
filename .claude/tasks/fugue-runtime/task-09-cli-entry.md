# Task 09 — CLI Entry Point

## Outcome

`packages/fugue/runtime/src/index.ts` is a runnable CLI script. `node packages/fugue/runtime/src/index.js --task "Build X" --repo "owner/repo" --issue 42` validates env vars, parses arguments, and calls `runFugue()`. Missing required args or env vars produce a clear error message and exit code 1.

## Why

The CLI is how both humans and GitHub Actions trigger the Fugue runtime. It is the public interface. Without it, `runFugue()` can only be called programmatically from other TypeScript files.

## Context

### Repository state

Task 08 is complete. `packages/fugue/runtime/src/orchestrator.ts` exports `runFugue(taskDescription, repo, issueNumber)`.

### Arguments

```
--task    Required. The task description passed to /breakdown.
--repo    Required. GitHub repo in "owner/repo" format.
--issue   Required. GitHub issue number that triggered this run (for posting comments).
```

### Required environment variables

```
ANTHROPIC_API_KEY
GITHUB_TOKEN
```

`LANGCHAIN_API_KEY` and `LANGCHAIN_TRACING_V2` are optional — tracing is enabled when present.

### Implementation

```typescript
#!/usr/bin/env node
import { runFugue } from './orchestrator.js';

function parseArgs(): { task: string; repo: string; issue: number } {
  const args = process.argv.slice(2);
  // ... parse --task, --repo, --issue flags
}

function validateEnv(): void {
  const required = ['ANTHROPIC_API_KEY', 'GITHUB_TOKEN'];
  const missing = required.filter(k => !process.env[k]);
  if (missing.length > 0) {
    console.error(`Missing required env vars: ${missing.join(', ')}`);
    process.exit(1);
  }
}

async function main(): Promise<void> {
  validateEnv();
  const { task, repo, issue } = parseArgs();
  await runFugue(task, repo, issue);
}

main().catch(err => {
  console.error(err);
  process.exitCode = 1;
});
```

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/index.ts` | Replace empty barrel with CLI implementation |

## Out of Scope

- Do not add a config file format (JSON, YAML) — args and env vars only.
- Do not add `--help` output — keep it minimal.

**Dependency:** Task 08 must be complete (`runFugue` must exist and be importable).

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes.
2. `node packages/fugue/runtime/src/index.js` (no args) exits with code 1 and prints a clear error about missing `--task` and `--repo`.
3. Running without `ANTHROPIC_API_KEY` set exits with code 1 and names the missing variable.
4. `node packages/fugue/runtime/src/index.js --task "test" --repo "owner/repo" --issue 1` with valid credentials begins the Fugue workflow (calls `runFugue`).
