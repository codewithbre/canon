# Architecture

## Agent harness family

Orchestra's agents are built on a set of standalone, agnostic harnesses. Each
harness has a single contract and can be used independently of Orchestra or
any other agent.

| Harness | Contract |
|---|---|
| [plan-to-pr](https://github.com/codewithbre/plan-to-pr) | intent → task documents |
| [ai-eval-agent](https://github.com/codewithbre/ai-eval-agent) | output → eval report |

Harnesses are agnostic to each other. plan-to-pr has no opinion on how task
documents are implemented or tested. ai-eval-agent has no opinion on how the
output it evaluates was produced. This is intentional. Working across different points in the pipeline requires
portable harnesses that compose into any agentic system or workflow, not ones
that are coupled to a specific agent.

---

## Execution modes

### Interactive (Claude Code/Cursor)

The human invokes each skill manually, one at a time or via a skill router, in
a coding session. The skill chain runs sequentially with the human at each
checkpoint.

```
Human: /breakdown
  → agent produces task list, confidence ratings, overview.md
Human: /write-task (once per approved task)
  → agent produces task documents
Human: /verify-task
  → agent produces gap report
Human: /create-tracker
  → agent produces IMPLEMENTATION.md
Human: /implement (per task, per session)
  → agent produces committed code
Human: /create-pr
  → agent produces PR description
```

### Programmatic/Autonomous workflows (Octave runtime)

The Octave runtime runs the planning chain automatically, without manual skill
invocations. It calls the Claude API directly, manages GitHub, and waits at
two human gates.

```
GitHub issue (intent)
  → Octave: load scaffold files from target repo
  → Octave: run breakdown
  → [Gate 1: human approves breakdown via 👍 or "approved" comment]
  → Octave: fan-out write-task (parallel, N workers)
  → Octave: fan-out verify-task (parallel, N workers)
  → Octave: commit task documents, open PR
  → [Gate 2: human merges PR]
  → Octave: run reprise, post retrospective issue
```

The two gates are the only points where a human must act. Everything between
them runs autonomously.

---

## Octave runtime internals

`packages/octave/runtime/src/`

### Orchestration

`orchestrator.ts` implements the full planning pipeline. Key patterns:

**Sequential chain**: top-level steps run in order. Each step's output is
the next step's input.

**Parallel fan-out**: write-task and verify-task run concurrently across all
tasks. `fanOut()` processes items in configurable chunks with retry logic:

```ts
limits: {
  concurrency: 4,     // parallel workers per chunk
  maxRetries: 3,      // retries before marking as failed
  interChunkDelayMs: 500
}
```

**Failure tracking**: failed workers are collected in `ctx.failures`, never
silently dropped. If any failures or LOW-confidence verify results exist, Octave
posts a gap report and stops before committing.

**Dependency ordering**: `topologicalSort()` orders task documents by their
`dependsOn` fields before committing, ensuring the PR presents tasks in safe
execution order.

### Human gates

**Gate 1**: Octave posts the task breakdown as a GitHub issue comment and
polls for a 👍 reaction or an "approved" comment before proceeding.

**Gate 2**: Octave polls for the task-doc PR to be merged before running
reprise.

### Context scoping

Scaffold files (README.md, CLAUDE.md, package.json) are loaded once from the
target repo and passed as `codebaseSummary` to each write-task worker. Each
worker receives the summary plus one task, nothing else.

### Prompt caching

Skill prompts are passed as cached system message blocks:

```ts
new SystemMessage({
  content: [{
    type: 'text',
    text: skillContent,
    cache_control: { type: 'ephemeral' },
  }],
})
```

With N parallel write-task workers, the skill prompt is paid for once and
read at ~10% cost on every subsequent call.

### Model selection

| Step | Model | Reason |
|---|---|---|
| breakdown | `claude-sonnet-4-6` | Highest ambiguity: requires reasoning |
| write-task | `claude-sonnet-4-6` | Context synthesis across codebase and task |
| verify-task | `claude-haiku-4-5-20251001` | Structural check: does the doc meet criteria |
| reprise | `claude-haiku-4-5-20251001` | Formatting structured run data into a report |

---

## What stays human

The orchestrator handles the mechanics of orchestration and implementation.
Humans handle judgment and approvals. This does not change regardless of how
much of the chain is automated.

- Approving the breakdown before task documents are written
- Reviewing task documents before the implementation PR is merged
- UAT after implementation, did the outcome match the intention
- Architectural decisions surfaced as flags during breakdown
- Resolving ASK AUTHOR findings from verify-task

---

## Planned agents

| Package | Role | Status |
|---|---|---|
| `packages/octave` | Build agent: planning through task docs | Active |
| `packages/coda` | Test and deploy agent | Under Development |
