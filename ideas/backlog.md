# Backlog — Orchestra / Octave

Items captured from the POC session (Goals 1, 2, 3). Address in priority order.
Each item notes where it was observed.

---

## 1. Feedback loops — highest priority

These are gaps in the current Gate 1 / Gate 2 model that assume perfect forward
progress. Real usage will have concerns, rejections, and change requests.

### Gate 1 revision loop
*Observed: all three POC runs — no path existed for rejection or revision*

Currently Gate 1 only polls for `approved` or 👍. Any other comment is ignored.

Needed:
- `👎` or comment containing `revise:` → re-run breakdown with the comment
  appended to the intent as additional context
- `stop` or `abort` → cancel the run, post a closing comment
- Any other comment → Octave acknowledges it but continues polling (currently
  it just silently ignores it)

### Gate 2 revision loop
*Observed: all three POC runs — PR "changes requested" is invisible to Octave*

Currently Gate 2 only polls for PR merge. A "Changes requested" review blocks
forever.

Needed:
- Poll PR review state alongside merge state
- If `changes_requested`: read review body, re-generate the affected task doc
  with the review as additional context, push a new commit to the PR branch,
  post a comment explaining what was changed
- If PR is closed without merge: abort with a clear message on the issue

---

## 2. Breakdown skill improvements

### Parallel tasks — breakdowns produce linear chains only
*Observed: Goals 1, 2, 3 — every breakdown was a fully sequential chain*

No run produced tasks with parallel execution potential. Real projects have
independent work. The `/breakdown` skill should:
- Explicitly look for tasks with no shared dependencies
- Group them as parallelizable in the task list
- The orchestrator's fanOut already supports this — the breakdown just needs
  to produce the right dependency graph

### Error handling never surfaces
*Observed: Goal 3 reprise — MCP server had no error handling task or criteria*

No breakdown ever produced an error handling task or added edge-case acceptance
criteria. For any tool, library, or server task, breakdown should explicitly ask:
- What happens on invalid input?
- What happens on boundary conditions (division by zero, null, overflow)?

Add to `/breakdown` skill: "For each task involving I/O, computation, or external
calls, surface error handling as either a dedicated task or as acceptance criteria."

### Arbitrary task splitting
*Observed: Goal 3 — add/subtract and multiply/divide split across two tasks*

Tasks 03 and 04 in Goal 3 were four identical function registrations — one PR's
worth of work. The breakdown split them to manage task count, not for a technical
reason. Add guidance to `/breakdown` skill: "Tasks should represent meaningful
scope boundaries, not arbitrary split points. Four identical operations is one task."

---

## 3. verify-task improvements

### Shallow structural check — not semantic
*Observed: verify-task speeds of 1-2s per doc suggest surface-level checking*

Currently verify-task checks that sections are present and placeholders are removed.
It doesn't check whether the task doc is actually implementable by an isolated agent.

Needed: push verify-task to check:
- Does the Context section contain everything the agent needs to start cold?
- Are acceptance criteria objectively verifiable (not "code should work correctly")?
- If the task depends on a prior task, does it restate what that task produces?
- Are file paths concrete, not vague ("a module" vs `src/operations.ts`)?

---

## 4. Runtime improvements

### Testing — never in scope
*Observed: all three POC runs — no breakdown ever produced a test task*

Add `--with-tests` flag to the CLI. When set, the breakdown prompt instructs
it to include a test task for each implementation task. Default: off.

### Reprise → skill update loop
*Observed: reprise produces good suggestions but nothing acts on them*

The improvement cycle is currently: reprise identifies gap → human reads it →
maybe updates skill → maybe improves. There's no automated path.

Option: after reprise posts the issue, Octave could open a follow-up PR to
`orchestra/skills/` with proposed skill updates based on the Suggested
Improvements section. This would be a new tool: `runSkillUpdate()`.

### Log file should be structured JSON
*Observed: octave.log is human-readable text — hard to parse for metrics*

Add structured JSON log alongside the human-readable log:
`--log-file octave.log` → human text
`--json-log octave.json` → structured per-step events with timestamps and tokens

This makes it easy to build tooling on top of the log data.

---

## 5. Attribution — bot account

*Observed: all runs — issues/PRs appear as codewithbre, not a Octave agent*

`FUGUE_GITHUB_TOKEN` env var is wired in `github.ts` (`getBotOctokit()`).
The token just needs to be set.

Options:
- Create a GitHub account `octave-bot`, generate a PAT, set as `FUGUE_GITHUB_TOKEN`
- Create a GitHub App for more official `[bot]` attribution

---

## 6. Process and skills

### /onboard-project before writing CLAUDE.md
*Observed: I wrote the calculator CLAUDE.md from assumptions and got the stack wrong*

Convention: always run `/onboard-project` on the target repo before the first
Octave run. Add to CLAUDE.md of Orchestra: "Before running Octave on a target repo,
ensure a CLAUDE.md exists. If none exists, run /onboard-project first."

### Claude Desktop compatibility acceptance criteria
*Observed: Goal 3 reprise — Task 06 had vague acceptance criteria*

For any MCP server task, acceptance criteria must include a concrete verification
step: "server starts without errors when invoked with `node dist/index.js`" and
"MCP protocol handshake completes when client connects."

### verify-task API variant
*Observed: verify-task currently uses withStructuredOutput so it works via API,
but the skill prompt has "spawn a task-doc-verifier agent" instruction*

Test whether this causes issues in practice. If the agent-spawning instruction
produces unexpected output via API, create a `verify-task-api.md` variant.

---

## 7. Cxt-manager integration

*See ideas/session-rate-monitoring.md and ideas/dashboard-and-observability.md*

After the first real Octave run (Goals 1-3 POC complete), build cxt-manager v2
and wire it into Orchestra at the natural breakpoints:
- After breakdown: `cxt commit "breakdown complete — N tasks queued"`
- After each Gate merge: `cxt commit "task X merged, next is Y"`
- At session end: `cxt commit "session ended — stopped at step Z"`

This solves the "where are we" problem without a STATUS.md.

---

## 8. GitHub Actions trigger

*Observed: Actions workflow exists but packages/ is gitignored — runtime not available in CI*

The `.github/workflows/octave.yml` exists but can't run because the runtime source
is private. Options to fix:
- Publish `octave-runtime` to npm (then `npx octave` works in CI)
- Make `packages/octave/runtime/src/` public (breaks the privacy model)
- Create a separate private runtime repo that Actions checks out

Lowest friction: publish to npm under a scoped package (`@codewithbre/octave`).
The public repo would then have a working Actions trigger.

---

## 9. Coda

*See CLAUDE.md — "Coda is not yet scoped. Do not scope it unless asked."*

Coda (test + deploy agent) should be scoped after:
1. At least one full Octave run produces code that's actually implemented by a human
2. The implemented code needs testing and deployment
3. The handoff format from Octave (task docs, PRs) is stable

Do not scope Coda until then.
