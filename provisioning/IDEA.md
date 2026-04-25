# Avangarde Harness — Idea

## What this is

A personal OpenSpec model for agent-assisted development.
Five scripts, each wizard-driven. Human validates at the end of each phase.

---

## Mental model

```
init-project.sh     →  Wizard: what to build, tech stack  →  Human approves constitution
plan-add-scope.sh   →  Wizard: plan phases OR add scope   →  Human approves task list
execute-pre-task.sh →  Explains BDD, checks sensors       →  Human approves before coding
execute-task.sh     →  Ralph loop until sensors pass
closure-task.sh     →  Human validates behavior           →  Commit + archive
```

Each task lives on its own branch. Commits are small and focused.

---

## The scripts

### `init-project.sh`
Constitution phase. Run once per project.

Wizard prompts:
- "What are you building?"
- "What tech stack? (or let AI suggest based on description)"
- "Any constraints or non-goals?"

AI proposes: vision, data model, architecture rules, initial guides.
Human approves or edits.

Output: `guides/` seeded, `specs/product/vision.md` created.

---

### `plan-add-scope.sh`
Planning + scope management. Two modes, one script.

**Mode 1 — Plan** (first run or re-plan):

Wizard flow:
- AI reads constitution and proposes a phased task list.
- Walks human through each phase: "Phase 1 covers X. Add, remove, or OK?"
- Human approves phase by phase.

**Mode 2 — Add scope** (new feature or bug fix mid-project):

Wizard flow:
- "What needs to be done?"
- "When does it need to be done? (now / next phase / backlog)"
- AI assesses dependencies, proposes placement, flags if it preempts current work.
- Human approves placement.

Output: `harness/loops/tasks.md` created or updated.

**Task list shape:**
```
## Phase 1 — Foundation
[ ] DB schema + migrations
[ ] Environment config

## Phase 2 — Backend (requires Phase 1)
[ ] API: list recipes
[ ] API: get recipe
[ ] API: create recipe

## Phase 3 — Frontend (requires Phase 2)
[ ] UI: recipe list
[ ] UI: recipe detail
[ ] UI: create/edit form
```

---

### `execute-pre-task.sh`
Pre-task phase. Run once per task before any coding.

Wizard flow:
1. Picks next unchecked task from `tasks.md`.
2. AI generates BDD acceptance criteria and explains them to the human.
3. **Sensor check**: does a sensor exist for this task type?
   - If yes: confirm it covers this task.
   - If no: scaffold the sensor first (e.g. API task → add API test template, UI task → add component test template).
4. Human approves BDD and sensors before coding starts.
5. Creates a dedicated branch for this task.

Output: spec file with BDD, branch created, sensors confirmed.

---

### `execute-task.sh`
Implementation phase. Ralph loop.

1. AI codes against the approved BDD on the task branch.
2. Runs sensors on every iteration (lint, typecheck, tests, structural checks).
3. If the same failure repeats twice: stops, surfaces to human, asks to improve guides or spec before continuing.
4. Done when: all sensors green.

At natural checkpoints, calls `commit.sh` to keep commits small and focused.

---

### `closure-task.sh`
Closure phase. Run when sensors are green.

Wizard flow:
1. Diffs output against original BDD criteria.
2. Presents to human: "Does the behavior match the intent?"
3. On approval:
   - Calls `commit.sh` for final commit.
   - Marks task done in `tasks.md`.
   - Archives spec.
   - Merges branch.
4. On rejection: returns to `execute-task.sh` with human notes.

---

### `commit.sh`
Used by `execute-task.sh` and `closure-task.sh`. Not called directly.

1. Auto-detects changed files.
2. AI suggests a short, focused commit message based on the diff.
3. Human approves or edits.
4. Commits only files relevant to the current task (no accidental noise).

Rules enforced:
- One concern per commit.
- No commit touches files outside the current task scope without explicit approval.
- Message format: `<type>: <short description>` (e.g. `feat: add recipe list endpoint`).

---

## What the harness contains

```
harness/
  guides/         ← what the agent reads before coding
    AGENTS.md
    architecture.md
    coding-rules.md
  sensors/        ← automated validation, grown incrementally per task type
    check.sh
  loops/          ← task list and loop state
    tasks.md
    progress.md
specs/
  product/        ← vision, data model, non-goals
  features/       ← one file per task, includes BDD acceptance criteria
```

Sensors are not defined upfront in bulk. They grow as task types are encountered in `execute-pre-task.sh`.

Coding style rules live in `guides/coding-rules.md`, file-type specific:
- `.md` → clean, concise, no bloat
- `.jsx` → clean separation of elements
- `.test.js` → one assert per test, prioritize coverage

---

## Git discipline

- Every task runs on its own branch.
- Commits are small: one concern, no unrelated changes.
- `commit.sh` enforces this by scoping to changed files and surfacing AI-suggested messages for human approval.
- Branch merged only after `closure-task.sh` human approval.

---

## Harness concepts

| Concept | Role |
|---|---|
| **Spec / OpenSpec** | Describes intent, constraints, and what done means |
| **Harness** | Guides + sensors + loop state that make outcomes repeatable |
| **Ralph loop** | Repeated AI execution cycle against explicit acceptance checks |

The key mindset shift:

- One-off mistake → fix the code
- Repeating mistake → fix the harness (guides, spec, or checks)

---

## Non-goals (MVP)

- No multi-agent orchestration yet
- No GitHub issue sync yet
- No auto-generated review prompts
- No memory of prior harness patches

These come after the five-script loop works end to end.
