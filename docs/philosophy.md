# Principles of Orchestra

## Constraints make AI reliable

A less capable model given a precise, scoped, verifiable task will consistently
outperform a smarter model given a vague prompt. The bottleneck in AI-assisted
development is almost never the model's intelligence. It is the quality of the
instructions.

---

## Delegation, not micromanagement

Humans cannot micro-manage agentic systems without becoming a bottleneck to them.
To delegate means to set expectations and verify that the work is being done as
it should.

The plan is where human judgment lives, as an upfront decision about what success
looks like. The task document is the delegation contract: it captures what the
human decided, what the agent is responsible for, and how to verify the handoff
worked. Once that contract exists, the agent executes against it autonomously.
The human comes back at established checkpoints to verify the outcome, not to
supervise the steps.

This is why agents in Orchestra are designed to be as simple as possible and
agnostic to each other. The intelligence and judgment needed to complete the task
belongs in the task document, not the model or session context. By the time a
model executes, there should be no ambiguity left to resolve.

---

## Task Integrity

The primary principle. Every unit of work must be a single functional change with:

- **Clear intent**: what will be different when this is done
- **Clear location**: exactly which files are in scope
- **Clear output**: what the deliverable looks like
- **Negative framework**: what must not be done
- **A scope that can be verified in one pass**: if you cannot confirm it is done
  by reading the output, the task is too large or too vague

### Valid task

> Add a `getIntervalSemitones(interval: IntervalId): number` function to
> `packages/theory/src/intervals.ts` that returns the fixed semitone count for
> a given interval by looking it up in `INTERVAL_DEFINITIONS`. Export it from
> `src/index.ts`.

Clear intent, clear location, clear output, verifiable acceptance criterion.

### Invalid task

> Improve the intervals module.

This requires the agent to make scope, architecture, and design decisions that
belong to the human. Two competent developers would produce completely different
results. The task is not a task, it is a project.

---

## Why structure reduces model dependency

A task document that specifies the exact files, function signatures, acceptance
criteria, and explicit out-of-scope items does work the model would otherwise
have to infer. This has two practical consequences.

First, it makes smaller, cheaper models viable for execution. The model capability
you actually need scales with how much ambiguity you hand the model. Reduce the
ambiguity, reduce the model requirement.

Second, it makes the workflow model-agnostic. If your system depends on model
intelligence to fill in gaps, it breaks as models change. If it depends on
structured task documents, any model that can follow instructions can execute it.

---

## The skill chain enforces the philosophy

Each skill in the Orchestra workflow encodes a specific constraint:

| Skill | Constraint it enforces |
|---|---|
| `/workflow` | No work begins without a plan; no plan proceeds without human approval |
| `/breakdown` | No implementation begins until work is decomposed into verifiable tasks |
| `/write-task` | No task is executed without a self-contained document |
| `/verify-task` | No document proceeds to implementation if a cold agent could not execute it |
| `/create-tracker` | No implementation begins without visible sequencing and progress tracking |
| `/implement` | No changes outside the task document's defined scope |
| `/create-pr` | No merge without a documented, reviewable PR |
| `/feedback` | No scope expansion mid-task; gaps return to the task document, not to the agent |
| `/reprise` | No run ends without a retrospective that feeds back into skill improvement |

Together they make it structurally difficult for an agent to do unauthorized work,
guess at scope, or proceed past an ambiguous requirement without surfacing it.

