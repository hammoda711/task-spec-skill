# task-spec

A Claude Code skill that turns a task description into a written spec before any code is written.

It splits the description into numbered rules, asks you about the gaps instead of guessing at them, records every answer as a dated note on the rule it belongs to, and writes acceptance criteria before the implementation exists. It writes the spec and stops. It does not implement.

## Install

```
/plugin marketplace add hammoda711/claude-skill-task-spec
/plugin install task-spec@claude-skill-task-spec
```

## Use

Run `/task-spec` with the task, or just hand Claude a ticket. The skill also triggers on "new task", "add this task", "put it in tasks", and "the requirement changed".

```
/task-spec when a proposal is approved, set approved_at, and clear it if the status changes
```

## What it does

1. **Finds the file.** One file per subject under `context/tasks/`, named by subject in kebab-case — `proposal-status.md`, never `task1.md`. An existing file on the same subject is edited, not duplicated.
2. **Inspects the code first.** Reads the affected models, endpoints, hooks and constraints, so questions the codebase can already answer never get asked.
3. **Divides the description into parts.** One rule per part — a part that needs the word "and" to join two behaviours gets split. Your wording is kept where it is already precise.
4. **Asks about the gaps.** Batched, at most four at a time, each with real options. The reverse transition, the repeat call, who may do it, the error a refusal gives, existing rows, and which layer owns the rule are the ones that usually matter.
5. **Writes the file** in six fixed sections: Current requirement, Open questions, Acceptance criteria, Where it lives, Not done, Verified.
6. **Shows you the spec and waits.** It never starts building on its own.

Unanswered questions are classified **Blocking** or **Non-blocking** and stay in the file. A gap you did not answer is written `_Not yet defined._`, never filled with a guess that reads like a requirement.

`Verified` holds measurements only, added after something has actually been run. It is never pre-filled.

## Assumes

- **A `context/tasks/` folder** in the project. The skill creates it if it is missing.
- **That `context/` is yours and gitignored.** The spec files are working notes, not repository documentation. If you want them committed, that is your call to make in your own `.gitignore`.
- **A `context/context-change-log.md`** to append to. The skill logs each spec it writes there. Without the file it has nowhere to log.
- Nothing else. No language, framework or VCS requirement.

The examples in the skill come from a Python, FastAPI and SQLAlchemy codebase. The method does not depend on that stack.

## License

MIT. See [LICENSE](LICENSE).
