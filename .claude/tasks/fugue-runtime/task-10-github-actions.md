# Task 10 — GitHub Actions Workflow

## Outcome

`.github/workflows/fugue.yml` triggers when a GitHub issue in any repo using this workflow is labeled `fugue`. It installs the runtime dependencies and runs the CLI with the issue title as `--task` and the labeling event's repo and issue number as `--repo` and `--issue`. All required secrets are wired in.

## Why

The GitHub Actions workflow is what makes Fugue operable without a human at a terminal. Any project that copies this workflow file and configures the required secrets can trigger Fugue simply by labeling an issue.

## Context

### Repository state

Task 09 is complete. `packages/fugue/runtime/src/index.js` (compiled from `index.ts`) is runnable via `node`.

Note: the runtime uses TypeScript source run via `node --env-file=.env` with tsx, or compiled to JS. The workflow must handle whichever approach is used. Recommend using `tsx` (already in the ecosystem) to run TS directly:

```bash
npx tsx packages/fugue/runtime/src/index.ts --task "..." --repo "..." --issue "..."
```

This avoids a compile step in CI.

### Workflow

```yaml
name: Fugue

on:
  issues:
    types: [labeled]

jobs:
  run:
    if: github.event.label.name == 'fugue'
    runs-on: ubuntu-latest
    timeout-minutes: 60

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install pnpm
        uses: pnpm/action-setup@v4

      - name: Install dependencies
        run: pnpm install --filter fugue-runtime

      - name: Run Fugue
        run: |
          npx tsx packages/fugue/runtime/src/index.ts \
            --task "${{ github.event.issue.title }}" \
            --repo "${{ github.repository }}" \
            --issue "${{ github.event.issue.number }}"
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          LANGCHAIN_API_KEY: ${{ secrets.LANGCHAIN_API_KEY }}
          LANGCHAIN_TRACING_V2: 'true'
          LANGCHAIN_PROJECT: 'fugue'
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Required secrets in the target repo

- `ANTHROPIC_API_KEY` — from Anthropic console
- `LANGCHAIN_API_KEY` — from smith.langchain.com (optional, for tracing)
- `GITHUB_TOKEN` — automatically provided by Actions

## Scope

| File | Action |
|---|---|
| `.github/workflows/fugue.yml` | Create at monorepo root |

## Out of Scope

- Do not add the `tsx` dependency — use `npx tsx` which resolves from the workspace.
- Do not add a cron schedule — issue label trigger only.
- Do not create repo-specific secrets — those are configured by whoever adopts this workflow.

**Dependency:** Task 09 must be complete.

## Acceptance Criteria

1. `.github/workflows/fugue.yml` exists at the repo root.
2. Opening an issue on `codewithbre/orchestra` with label `fugue` and title "Test run" triggers the workflow (visible in Actions tab).
3. The workflow run logs show the Fugue CLI starting and printing the task description and repo.
4. The workflow exits 0 if the CLI exits 0, and 1 if the CLI fails.
