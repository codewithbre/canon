# Token Efficiency

## The problem

Canon's sequential skill chain wastes tokens in predictable ways. Every `/write-task` invocation re-loads the skill prompt and re-reads the same codebase files, even when the previous invocation just read them. For 10 task docs, that's 10x the reads and roughly 5,000 tokens of redundant skill-prompt overhead.

These are the observed patterns from using the manual skill chain in practice.

---

## Where token budget goes

### 1. Sequential /write-task (biggest waste)

Each invocation:
- Loads the full skill prompt (~500 tokens)
- Re-reads codebase files relevant to the task (varies, but often the same files across tasks)
- Produces one document

For 10 tasks sharing the same source files: ~10x the file reads, ~5,000 tokens of duplicate skill prompts.

**Fix:** Batch all write-task calls in one message — pass the codebase context once, produce N documents. The skill format is what matters, not the invocation count.

### 2. Re-reading files already in context

Files like `src/index.ts`, `package.json`, and `tsconfig.json` get read multiple times across skill invocations even when they haven't changed.

**Fix:** One broad Read pass at session start. Reference the loaded content throughout. Don't re-read unless the file changed.

### 3. Sequential /verify-task

Each task doc can be verified independently. Running 10 verifiers sequentially when they could run in parallel wastes wall-clock time proportional to N.

**Fix:** Pass the full tasks directory to `/verify-task` in one call. One context load, all docs verified sequentially by the same agent.

### 4. No persistent context across sessions

The IMPLEMENTATION.md tracker solves the human handoff problem, but agents re-derive codebase context at the start of each session. Stable files (tsconfig, package.json, engine source) are prime candidates for Anthropic's prompt caching.

---

## What the runtime fixes

The programmatic runtime (see `docs/architecture.md` and `runtime/`) addresses all four:

1. **Batched write-task** — orchestrator reads shared files once at breakdown time, passes extracted context to each parallel worker
2. **No redundant reads** — orchestrator holds context in memory across worker calls
3. **Parallel verify** — orchestrator fans out N verification calls simultaneously
4. **LangSmith tracing** — every API call is traced with input tokens, output tokens, cost, and latency — no more estimation

---

## Current best practice (manual mode)

Until the runtime exists, the most efficient manual approach is:

1. `/breakdown` → get the task list
2. In one message: "Write task docs for tasks 1 through N, save to `.claude/tasks/`" — Claude writes all N docs without separate invocations
3. `/verify-task .claude/tasks/` — one call, whole directory
4. `/create-tracker .claude/tasks/` — one call
5. Hand off to Cursor for `/implement` — one task per session

Steps 2-4 in a single Claude session, front-loaded before handing off to implementation.
