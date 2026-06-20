# Architecture

## Current: Sequential single-agent

Today, canon runs as a sequential workflow through one Claude Code instance. The human invokes each skill manually, one at a time:

```
Human invokes /breakdown
    → agent produces task list
Human invokes /write-task (×N, once per task)
    → agent produces N task docs
Human invokes /verify-task
    → agent produces gap report
Human invokes /create-tracker
    → agent produces IMPLEMENTATION.md
Human invokes /implement (per task, per session)
    → agent produces branch + PR
```

This works and produces reliable results. The cost is time — writing N task docs sequentially means N skill-load cycles and N codebase context re-derivations, even when the same files are relevant to multiple tasks.

---

## Where token and time efficiency is lost

**1. Sequential /write-task**
Each invocation loads the full skill prompt and re-reads the codebase files relevant to that task. For 10 tasks that share the same source files, that's 10x the reads. A parallel approach would read files once at breakdown time and pass the extracted context to each write-task agent.

**2. Sequential /verify-task**
Each task doc can be verified independently. Running 10 verifiers sequentially when they could run in parallel wastes wall-clock time proportional to N.

**3. No persistent context across sessions**
The IMPLEMENTATION.md tracker solves the session-handoff problem for humans, but agents still re-derive codebase context at the start of each session. Prompt caching (Anthropic's cache_control mechanism) would help here for stable files.

---

## Future: Orchestrator-worker fan-out

The sequential model is the manual version of what an orchestrator-worker system would automate:

```
Orchestrator
    └── runs /breakdown → task list
    └── fans out: N parallel write-task workers
          ├── Worker A: task-01.md
          ├── Worker B: task-02.md
          └── ...Worker N: task-N.md
    └── fans out: N parallel verify workers
          ├── Verifier A: task-01.md
          └── ...Verifier N: task-N.md
    └── runs /create-tracker
    └── sequential: implement workers (respecting dependency order)
          ├── Worker A: task-01 → PR
          └── Worker B: task-02 → PR (after task-01 merges if dependent)
```

The Agent tool in Claude Code already supports background agents with `run_in_background: true`. The infrastructure exists. The constraint today is subagent model availability and coordination overhead — both solvable.

### What the orchestrator owns

- Invoking `/breakdown` and extracting the structured task list
- Reading shared codebase files once and passing relevant context to each worker
- Spawning write-task workers in parallel
- Collecting their outputs and spawning verify workers in parallel
- Running `/create-tracker` after all verifications pass
- Spawning implement workers sequentially, respecting task dependencies
- Tracking completion and surfacing failures

### What stays human

- Approving the breakdown output before workers are spawned
- Reviewing PRs before merging
- Resolving `ASK AUTHOR` findings from verify agents
- Making architectural decisions that appear as flags in the breakdown

The orchestrator handles mechanics. Humans handle judgment. That boundary does not change in the multi-agent model.

---

## Relationship to existing tools

Canon is not a replacement for CI/CD, test frameworks, or deployment pipelines. It is a layer that sits between "human has an idea" and "code exists in a PR ready for review."

```
Human idea
    └── Canon workflow → PR
                            └── CI/CD → Merge → Deploy
```

Canon's output is always a PR. What happens after the PR merges is out of scope.
