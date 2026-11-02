---
name: task-spec
description: >
  Turn a task description into the spec file for it under context/tasks/. Breaks
  the description into numbered parts, asks about the gaps instead of guessing,
  and records each answer as a note on the part it belongs to. Use when the user
  hands over a new task, a requirement, a ticket or a feature description, when
  they say "put this in the tasks", or when a requirement on an existing task
  changes. Triggers on "/task-spec", "new task", "add this task", "put it in
  tasks", "the requirement changed".
---

# Task spec

Write the task down before writing any code. The file is the spec; the code follows it.

This skill **only writes the spec**. It does not implement, does not touch `app/`, does
not run migrations. If the user wants both, write the spec, show it, and let them say go.

---

## 1. Find the file

```bash
ls context/tasks/ 2>/dev/null || echo "(no tasks folder yet)"
```

Name the file by **subject**, kebab-case, never by number: `innovation-publish.md`,
`proposal-status.md`. `task1.md` tells a reader nothing in a month.

- **A file already covers this subject** — update that file. Do not create a second one.
  Changing a requirement means editing the part it belongs to, in place.
- **Nothing covers it** — create it.
- **Unsure whether it is the same subject** — ask. Two half-specs on one subject is worse
  than one long file.

If `context/tasks/` does not exist, create it. It lives inside `context/`, which is
gitignored, so nothing here reaches a commit or a PR.

### Inspect the existing implementation

Before dividing the requirements or asking questions, inspect the relevant existing code.

- Identify the affected models, schemas, endpoints, hooks, services, database constraints,
  and tests.
- Understand the current behavior without assuming it is the desired behavior.
- Use the actual implementation to answer questions that the code can settle.
- Identify technical claims that need verification.
- Do not modify application code or run migrations during this step.

Read the `context/` files that bear on the task as well — `api-contract.md`, `database.md`
and `decisions.md` — and treat a contradiction between them and the code as something to
report, not to silently resolve.

---

## 2. Divide the description into parts

Read the whole description first. Then split the **requirement** into numbered parts.

A part is one rule that can be true or false on its own, and that someone could verify by
itself. Split on behaviour, not on grammar:

> "published_at is filled only if available_on_network is true, else null, and the same
> for approved_at with the status"

is **two** parts, not one — they are two columns driven by two different conditions, and
either can be right while the other is wrong.

Rules for splitting:

- One rule per part. If a part needs the word "and" to join two behaviours, split it.
- Keep the user's own wording where it is already precise. Do not restate it in your own
  words to sound tidier — their phrasing is the requirement.
- A part that is an example, not a rule, goes under the rule it illustrates.
- Do not invent parts the user did not ask for. If you think one is missing, that is a
  question for step 3, not a part to add.

---

## 3. Ask about the gaps — do not guess

For each part, ask yourself what a developer would have to decide that the description
does not say. Only the ones that **change the work** become questions.

Ask with AskUserQuestion, batched — at most 4 at a time, and fewer is better. Each
question needs real options, not "what do you want".

Things that are almost always worth asking:

| Gap | Why it matters |
|---|---|
| What happens on the **reverse** transition | "set X when Y is true" rarely says what happens when Y goes false |
| **Repeat** of the same action | does the second call overwrite, hold, or refuse |
| Who may do it | an endpoint with no guard is a different task |
| The **error** a refusal gives | a status code is part of the contract |
| Existing rows | a new required field needs a backfill answer |
| Which layer owns it | a rule can sit in the schema, a hook, a route or a constraint, and the choice is the design |

Things **not** worth asking: anything with an obvious default, anything the codebase
already answers, and anything you can look up. Look first, ask second.

Record every answer as a note on the part it belongs to:

```markdown
2. `approved_at` holds a date only while `status` is `APPROVED`.
   **Note (2026-10-27):** on any other status the date is wiped, not kept. The column
   holds the current state, not a history. Asked and confirmed.
```

Date the note and say it was confirmed, so a later reader knows it is a decision and not
your assumption.

**A gap the user does not answer stays open.** Write `_Not yet defined._` under that part.
Never fill it with a guess that reads like a requirement.

### Classify unanswered questions

Classify each unresolved question:

- **Blocking:** The answer changes the required behavior or implementation and must be
  resolved before implementation.
- **Non-blocking:** The answer can be deferred without affecting the currently agreed
  scope.

Do not invent answers to blocking questions.

Record unresolved questions in the specification with their classification.

### Define acceptance criteria

For each requirement, define observable acceptance criteria before implementation.

Acceptance criteria must describe how to verify the expected behavior, including relevant:

- Successful operations.
- Invalid operations and expected errors.
- Reverse transitions.
- Repeated operations.
- Database state and API response consistency.

Do not invent behavior while writing acceptance criteria. Ask the user when a decision is
missing.

Acceptance criteria describe what must be tested. They are not evidence that testing has
passed.

Only record actual executed results in the `Verified` section.

---

## 4. Write the file

Use the following six sections, in this order. Keep them in this shape so every task file
reads the same:

```markdown
# <Subject>

<One line: what this task is for, in the user's terms.>

## Current requirement

1. <part>
2. <part>
   **Note (<date>):** <answer to a question that was asked>
3. <part>
   _Not yet defined._

## Open questions

- **Blocking** — <question>. Blocks part <n>.
- **Non-blocking** — <question>. Does not affect the agreed scope.

## Acceptance criteria

| # | Part | Given / When | Expected |
|---|---|---|---|
| 1 | <n> | <the call, and the state it starts from> | <status code, response field, row state> |

## Where it lives

| Thing | Where |
|---|---|
| <route / rule / column / migration> | <file, or symbol> |

## Not done

- <what is deliberately out of scope, and what it blocks>

## Verified

_Not yet implemented._
```

Notes on each section:

- **Current requirement** holds only what must be true **now**. Not history, not how it
  got here. If a requirement changes, edit the part; the change log records the change.
- **Open questions** carries every unresolved one with its classification, and names the
  part it blocks. An empty section is written as `_None._`, not deleted — a reader has to
  be able to tell "nothing open" from "nobody asked".
- **Acceptance criteria** says how each part will be checked, written before the code.
  Each row names the part it covers, so a part with no row is a part nobody planned to
  verify. These are intentions. They are **not** results.
- **Where it lives** identifies existing code confirmed by inspection, and the expected
  changes. Record actual file paths and symbols where known. Clearly distinguish verified
  existing locations from proposed changes. Update this section during implementation if
  the actual changes differ.
- **Not done** is for what is out of scope on purpose. Name what it blocks — "no approve
  route, so `approved_at` is unreachable over HTTP" is useful; "TODO: approve" is not.
- **Verified** is for measurements, with dates, and only after the thing has been run.
  Never pre-fill it. "Should work" never goes in it.

---

## 5. Log it, then show it

Add an entry to `context/context-change-log.md`, newest at the top, with
**Changed / Why / Impact**. Say which questions were asked and what was decided — the
answers are the part worth keeping.

Then show the user the file in chat: the numbered parts, the open questions with their
classification, and any `_Not yet defined._` lines. Ask whether to start building. Do not
start on your own.

### What approval does and does not authorize

- Approval of the specification authorizes implementation of the agreed scope.
- Unresolved **blocking** questions must be answered before implementation.
- Unresolved **non-blocking** questions do not prevent implementation if they do not
  affect the agreed scope.
- If implementation reveals a conflict with the approved specification, **stop and ask**
  before changing the requirements.

Approval is permission to build what the spec says. It is not permission to decide the
things the spec left open.

---

## When the requirement changes later

Same skill, same file:

1. Edit the part. Keep the number if the rule is the same rule changed; renumber only if
   parts were added or removed.
2. Add a dated note saying what it is now. Do not leave the old wording in place as if it
   were still true.
3. If the change invalidates something in **Verified**, strike that line — a measurement
   of the old behaviour is not evidence for the new one.
4. Log the change.

That last point matters for a reason worth stating exactly, because the obvious version of
it is wrong. A moving requirement does not quietly turn a correct claim into a wrong one.
What it does is make an **unmeasured** claim look load-bearing, so it gets repeated with
more confidence than it ever earned.

The guard is this. **Never state a technical claim as verified without evidence.** Use
executable tests or direct inspection of the relevant source code, as appropriate. Use
existing tests only when they demonstrate the exact behavior. If a claim cannot be
verified, label it as an unverified assumption and do not treat it as a requirement.
Record actual execution results in `Verified` only after running the relevant test or
operation.

**Relevant** is the load-bearing word. Inspecting the code that implements the behavior is
evidence; inspecting a similar-looking case elsewhere and reasoning by analogy is not.
That is the exact shape of the mistake: a claim about what `session.merge` does was
inferred from a different hook that looked the same, when the relevant source was the
service's `update` and merge's own rule about the instance `__dict__`. Reading the right
file would have settled it in a minute. Reading a neighbouring one produced a confident
wrong answer that survived several turns.
