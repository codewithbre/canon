# Philosophy

## The core claim

Constraints make AI reliable.

A less capable model given a precise, scoped, verifiable task will consistently outperform a smarter model given a vague prompt. The bottleneck in AI-assisted development is almost never the model's intelligence. It is the quality of the instructions.

Canon is built on that observation.

---

## Task Integrity

The primary principle. Every unit of work must be a single functional change with:

- **Clear intent** - what will be different when this is done
- **Clear location** - exactly which files are in scope
- **Clear output** - what the deliverable looks like
- **A scope that can be verified in one pass** - if you cannot confirm it's done by reading the output, the task is too large or too vague

### Valid task

> Add a `getIntervalSemitones(interval: IntervalId): number` function to `packages/theory/src/intervals.ts` that returns the fixed semitone count for a given interval by looking it up in `INTERVAL_DEFINITIONS`. Export it from `src/index.ts`.

This has a clear intent (new function), clear location (`intervals.ts`), clear output (a function that returns a number), and a verifiable acceptance criterion (call `getIntervalSemitones('perfect_5th')` and get `7`).

### Invalid task

> Improve the intervals module.

This requires the agent to make scope, architecture, and design decisions that belong to the human. Two competent developers would produce completely different results. The task is not a task — it is a project.

---

## Why judgment stays human

Agents are good at execution within defined bounds. They are not good at deciding what those bounds should be. Giving an agent the ability to decide scope produces unpredictable results — sometimes it does too little, sometimes too much, always in ways that are difficult to audit.

Canon keeps the decomposition step human-controlled:
- `/breakdown` produces a task list, but the human reviews and approves it
- `/write-task` documents each task, but the human confirms it before any agent implements it
- `/verify-task` checks that docs are complete, but does not modify them
- `/implement` executes within the scope defined in the task doc — nothing more

The workflow is designed so that every decision point where judgment matters is a human checkpoint.

---

## Why structure compensates for model limitations

A task document that specifies:
- The exact files to create or modify
- The exact function signatures
- The exact acceptance criteria
- Explicit out-of-scope items

...is a form of structured prompting that most agents can follow reliably regardless of their base capability. The structure does work that the model would otherwise have to infer.

This matters practically: the best model available today may not be the best model available when you return to a codebase in six months. If your workflow depends on model intelligence to fill in gaps, it breaks as models change. If your workflow depends on structured task documents, it is model-agnostic.

---

## The skill chain enforces the philosophy

Each skill in the canon workflow encodes a specific constraint:

| Skill | Constraint it enforces |
|---|---|
| `/breakdown` | No implementation begins until work is decomposed |
| `/write-task` | No task is executed without a self-contained doc |
| `/verify-task` | No doc proceeds to implementation if it has gaps |
| `/create-tracker` | No implementation begins without visible progress tracking |
| `/implement` | No changes outside the task doc's defined scope |
| `/create-pr` | No merge without a documented, reviewable PR |

Together they make it structurally difficult for an agent to do unauthorized work, guess at scope, or proceed past an ambiguous requirement without surfacing it.

That is the point.
