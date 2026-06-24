# Octave

The build agent. Takes intent, breaks it down into scoped tasks, writes task documents, verifies them, and implements each as a PR.

Part of the [Orchestra](../../README.md) monorepo.

## What it does

Octave executes the Orchestra skill chain:

```
/breakdown → /write-task → /verify-task → /create-tracker → /implement → /create-pr → /reprise
```

`/reprise` runs automatically at the end of every Octave run — posting a GitHub issue with what was accomplished, what shortcomings were found, and what skill improvements are suggested.

## Two execution modes

**Mode 1 — Claude Code (interactive):** Invoke skills manually in a Claude Code session. Human at the keyboard. Already works with the skill definitions in `orchestra/skills/`.

**Mode 2 — Programmatic API (automated):** The `runtime/` TypeScript orchestrator calls the Claude API directly. Enables GitHub Actions triggering, parallel fan-out, and LangSmith observability. Implementation in progress — 11 task docs written, see `.claude/tasks/octave-runtime/IMPLEMENTATION.md`.

## Docs

- [`docs/runtime-design.md`](docs/runtime-design.md) — how the runtime works, two modes, LangChain + LangSmith wiring
- [`docs/token-efficiency.md`](docs/token-efficiency.md) — where token budget goes and how to reduce it

See the root [`docs/`](../../docs/) for the philosophy and workflow that Octave is built on.
