# Idea: Token Efficiency Observations from Session 1

## What we measured

Prompt caching is working extremely well. Session 1 (153 min, 94 API calls):

- Cache reads: 7,851,274 tokens
- Cache creation: 101,582 tokens  
- Actual input tokens: 177 (near-zero — everything is cached)
- Output tokens: 58,975

The cache read/create ratio is ~77:1. The system prompt, CLAUDE.md, and
conversation context are being cached and reused almost perfectly.

## Implication for Octave runtime design

The token efficiency doc (`packages/octave/docs/token-efficiency.md`) identifies
sequential write-task as the biggest waste. The session 1 data confirms the
fix works: prompt caching collapses the per-call overhead to near-zero.

For the runtime, this means:
- The skill prompt caching strategy in task-03/04/05/06 is worth implementing
  even for small N (the overhead is real without it)
- The batching vs parallel-calls debate resolves clearly to parallel: with
  caching the prompt overhead is negligible, and parallel gives retry isolation

## Cache TTL observation

The `cache_creation` block distinguishes:
- `ephemeral_5m_input_tokens` — 5-minute cache writes
- `ephemeral_1h_input_tokens` — 1-hour cache writes (10,825 tokens this session)

The 1h cache is being used for the large stable system prompt block.
The 5m cache appears unused in this session (0 tokens).

For the Octave runtime: skill prompts (~500 tokens each) should use 5m TTL
since they're stable within a single orchestrator run but don't need 1h.
The shared context object (scaffold files, README) should also use 5m TTL.

## What to build

- [ ] Add cache TTL selection to the skill loader context: use `ephemeral_5m`
  for skill prompts and shared scaffold files within a single run
- [ ] Log `cache_creation_input_tokens` and `cache_read_input_tokens` per step
  in the orchestrator's token log (currently only logs input + output)
- [ ] Include cache hit ratio in the `/reprise` retrospective
