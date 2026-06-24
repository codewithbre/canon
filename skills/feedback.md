Capture a UAT finding and append a correction to the existing task document.
This skill has one responsibility: translate human judgment about an outcome
gap into a scoped correction appended to the task document. Do not implement
anything. Do not modify existing task document content. Do not append the
correction until the gap is confirmed.

Human judgment is front-loaded. The user articulates the gap before any
correction work begins. The implementing agent delivers the correction. It
does not interpret what the correction should be.

UAT finding: [describe what was observed vs. what was expected]

## Step 1: Capture the gap
Ask the user to describe:
- What the task was supposed to deliver (the original intention)
- What was observed during UAT (what actually happened)
- What needs to be different (the specific gap to close)

Present this back as:

  Intention: <what the task was supposed to deliver>
  Observed:  <what UAT revealed>
  Gap:       <what needs to be different>

STOP. Wait for the user to confirm the gap is accurately described.
This is the judgment gate. Do not proceed until confirmed.

## Step 2: Append the correction
Locate the existing task document for the implemented task.
Append the following section to the end of the document.
Do not modify, remove, or rewrite any existing content.
The original task document must remain fully intact.

---

## Correction

**UAT Finding**
What was observed that did not match the stated outcome.

**Gap**
What needs to be different.

**Corrected Outcome**
What will be true when the correction is delivered.

**Acceptance Criteria**
Observable, verifiable conditions that confirm the correction was made.

---

The correction is a discrete unit of work. The implementing agent reads
the full task document for context and the Correction section for scope.
The correction does not reopen or change the original task's outcome.
