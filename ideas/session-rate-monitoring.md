# Idea: Autonomous Session Rate Monitoring

## Problem

An autonomous agent running under a Claude Code Pro plan has no built-in command
to check remaining session quota. The "97% of session limit" signal comes from
the harness — not from anything the agent can query itself.

## What we found

Every assistant turn is written in real time to:

```
~/.claude/projects/<project-slug>/<session-id>.jsonl
```

Each entry has a full `usage` block:

```json
"usage": {
  "input_tokens": 3,
  "cache_creation_input_tokens": 10825,
  "cache_read_input_tokens": 15526,
  "output_tokens": 143,
  "service_tier": "standard",
  "speed": "standard"
}
```

Environment variables available inside the agent process:
- `CLAUDE_CODE_SESSION_ID` — current session UUID (links to the JSONL file)
- `CLAUDE_CODE_CHILD_SESSION` — 1 if sub-session
- `CLAUDE_CODE_ENTRYPOINT` — cli / ide / etc.

No `/status`, `/cost`, or `/usage` commands exist in Claude Code.
`service_tier` and `speed` are both `"standard"` at normal load — a shift
would indicate throttling, but this hasn't been observed in practice yet.

## Session numbers (this session, 153 min)

| Metric | Value |
|---|---|
| Unique API calls | 94 |
| Output tokens | 58,975 |
| Cache creation | 101,582 |
| Cache reads | 7,851,274 |
| Output tokens/hour | ~23k |

Pro throttle threshold is estimated around 32k output tokens/hour sustained.
This session ran at ~70% of that ceiling.

## Proposed utility

A `src/monitor.ts` module in the Octave runtime that an autonomous orchestrator
can call between tasks to estimate session pressure:

```typescript
type Pressure = 'low' | 'medium' | 'high';

// low  → proceed normally
// medium → watch, reduce parallelism
// high  → slow down, add inter-task delays, surface warning
function estimateSessionPressure(): Pressure {
  const usage = readSessionUsage(); // parses the JSONL
  const outputPerHour = usage.outputTokensPerMinute * 60;
  if (outputPerHour > 25000) return 'high';
  if (outputPerHour > 15000) return 'medium';
  return 'low';
}
```

The JSONL is slightly behind real-time — read it every N turns, not every turn.

## What to build

- [ ] `src/monitor.ts` — `readSessionUsage()` + `estimateSessionPressure()`
- [ ] Orchestrator calls `estimateSessionPressure()` before each `fanOut()`
- [ ] At `high` pressure: halve `ctx.limits.concurrency`, log a warning
- [ ] At session end, include pressure curve in the `/reprise` retrospective

## Open questions

- Does `service_tier` or `speed` actually shift before a hard stop, or does
  the session just terminate? Needs observation across multiple long sessions.
- Is the JSONL path stable across Claude Code versions, or does it move?
  The `CLAUDE_CODE_SESSION_ID` env var is the reliable anchor.
