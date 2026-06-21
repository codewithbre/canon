# Task 11 — Reprise Tool

## Outcome

`packages/fugue/runtime/src/tools/reprise.ts` exports `runReprise(repo: string, issueNumber: number, implementationMd: string): Promise<{ url: string }>`. It calls Claude with the reprise skill as system prompt, passes the IMPLEMENTATION.md content as context, and posts the resulting retrospective report as a GitHub issue labeled `reprise`. Returns the URL of the created issue.

The orchestrator calls this automatically as the final step after all tasks are merged.

## Why

The reprise gives the human a structured summary of what was accomplished, what gaps the skill chain exposed, and concrete suggestions for improving the workflow. Without it, every run ends silently — there is no feedback loop for improving the skills over time. Posting it as a GitHub issue makes it reviewable, linkable, and archivable.

## Context

### Repository state

Tasks 01-10 are complete. Additionally:
- `skills/reprise.md` exists at the monorepo root — this is the system prompt for the reprise call
- `src/skills.ts` exports `loadSkill('reprise')`
- `src/github.ts` exports a function for creating issues (add `createIssue` if not already present)
- `src/types.ts` has `VerifyResult` and all other types

### createIssue function

Add to `src/github.ts` if not already present:

```typescript
export async function createIssue(
  repo: string,
  title: string,
  body: string,
  labels: string[],
): Promise<{ url: string; number: number }> {
  const octokit = getOctokit();
  const { owner, repo: repoName } = parseRepo(repo);

  // Ensure label exists
  for (const label of labels) {
    try {
      await octokit.issues.getLabel({ owner, repo: repoName, name: label });
    } catch {
      await octokit.issues.createLabel({
        owner, repo: repoName, name: label,
        color: '8b5cf6',
        description: 'Post-run retrospective reports',
      });
    }
  }

  const response = await octokit.issues.create({
    owner, repo: repoName, title, body, labels,
  });

  return { url: response.data.html_url, number: response.data.number };
}
```

### Implementation

```typescript
export async function runReprise(
  repo: string,
  issueNumber: number,
  implementationMd: string,
): Promise<{ url: string }> {
  const llm = createLLM();
  const systemPrompt = loadSkill('reprise');

  const response = await llm.invoke([
    new SystemMessage(systemPrompt),
    new HumanMessage(
      `Generate the retrospective report for this completed run.\n\nIMPLEMENTATION.md:\n${implementationMd}\n\nTriggering issue: #${issueNumber} in ${repo}`,
    ),
  ]);

  const body = response.content as string;
  const title = `Reprise: ${extractOriginalGoal(implementationMd)}`;

  const { url } = await createIssue(repo, title, body, ['reprise']);
  return { url };
}

function extractOriginalGoal(implementationMd: string): string {
  // Extract the original goal from the first line or session notes
  // Fallback: return first 60 chars of the content
  const match = implementationMd.match(/^# (.+)/m);
  return match?.[1]?.slice(0, 60) ?? 'Completed run';
}
```

### Where the orchestrator calls this

In `src/orchestrator.ts`, after all tasks are confirmed merged, add:

```typescript
// Final step: reprise
const { url } = await runReprise(repo, issueNumber, implementationMdContent);
console.log(`[fugue] Reprise posted: ${url}`);
```

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/tools/reprise.ts` | Create — `runReprise()` |
| `packages/fugue/runtime/src/github.ts` | Update — add `createIssue()` if not present |
| `packages/fugue/runtime/src/orchestrator.ts` | Update — call `runReprise()` as final step |

## Out of Scope

- Do not modify the `skills/reprise.md` skill definition — it is read-only input.
- Do not read the IMPLEMENTATION.md from disk — it is passed in as a string by the orchestrator.

**Dependency:** Tasks 07, 08, 02, and 03 must be complete.

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes.
2. `runReprise('codewithbre/orchestra', 1, sampleImplementationMd)` creates a GitHub issue titled `Reprise: ...` with label `reprise` and returns its URL.
3. The issue body contains the required sections: Original Goal, What Was Delivered, What Worked Well, Shortcomings, Suggested Improvements, Metrics.
4. The LangSmith trace shows the reprise skill as the system prompt.
5. After a full `runFugue()` call completes with all tasks merged, a reprise issue is automatically created and its URL is logged to stdout.
