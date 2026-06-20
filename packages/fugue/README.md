# Fugue

The build agent. Takes intent, breaks it down into scoped tasks, writes task documents, verifies them, and implements each as a PR.

Part of the [Orchestra](../../README.md) monorepo.

## What it does

Fugue executes the canon skill chain:

```
/breakdown → /write-task → /verify-task → /create-tracker → /implement → /create-pr
```

## Two execution modes

**Mode 1 — Claude Code (interactive):** Invoke skills manually in a Claude Code session. Human at the keyboard. Already works with the skill definitions in `orchestra/skills/`.

**Mode 2 — Programmatic API (automated):** The `runtime/` TypeScript orchestrator calls the Claude API directly. Enables GitHub Actions triggering, parallel fan-out, and LangSmith observability.

## Docs

- [`docs/runtime-design.md`](docs/runtime-design.md) — how the runtime works, two modes, LangChain + LangSmith wiring
- [`docs/token-efficiency.md`](docs/token-efficiency.md) — where token budget goes and how to reduce it

See the root [`docs/`](../../docs/) for the philosophy and workflow that Fugue is built on.
