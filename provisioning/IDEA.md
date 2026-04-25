# Avangarde Harness — Idea

## What this is

A personal OpenSpec model for agent-assisted development.
Six scripts define six phases. Human validates at the end of each.

---

## Mental model

```
init-project.sh   →  AI proposes constitution (vision + stack + data model)  →  Human approves
plan-project.sh   →  AI proposes phased task list                            →  Human approves
add-scope.sh      →  AI places new feature/bugfix in correct phase           →  Human approves
new-change.sh     →  AI writes BDD for next task                             →  Human approves
run-loop.sh       →  AI codes (Ralph loop)  →  Sensors pass
archive-change.sh →  Diff against BDD  →  Human approves  →  Archived
```

The scripts ARE the phase boundaries. Everything else (guides, sensors, specs) are inputs and outputs to them.

---

## The six scripts

### `init-project.sh`
Constitution phase. Run once per project.

1. Human describes the product idea.
2. AI proposes: vision, tech stack, data model, architecture rules.
3. Human approves or edits.
4. Output: `guides/` seeded with architecture and coding rules, `specs/product/vision.md`.

### `plan-project.sh`
Planning phase. Run once after constitution, re-run when scope changes significantly.

1. AI reads the constitution and proposes an ordered, phased task list.
2. Tasks within a phase are independent and parallelizable.
3. Phases depend on prior phases completing.
4. Human approves the ordering and phase grouping.
5. Output: `harness/loops/tasks.md`.

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

### `add-scope.sh`
Scope change phase. Run when a new feature or bug fix arrives mid-project.

1. Human describes the new feature or bug fix.
2. AI assesses: what does it touch, what does it depend on, which phase it belongs to.
3. AI flags if it is a hotfix (preempts current work) or a backlog addition (queues into a phase).
4. Human approves placement and priority.
5. Output: task inserted into `tasks.md`, phase boundaries re-evaluated.

### `new-change.sh`
Specify phase. Run once per task.

1. AI picks the next unchecked task from `tasks.md`.
2. AI generates BDD acceptance criteria and scaffolds a spec file.
3. Human validates: "Does this BDD capture what done means?"
4. On approval: branch created, harness ready for loop.

### `run-loop.sh`
Implement phase.

1. AI codes against the approved BDD.
2. Ralph loop: iterate until sensors pass (lint, typecheck, tests, structural checks).
3. If the same failure repeats twice: stop, surface to human, improve guides or spec.
4. Done when: all sensors green.

### `archive-change.sh`
Review + archive phase.

1. Diffs output against original BDD criteria.
2. Prompts human: "Does the behavior match the intent? OK to archive?"
3. On approval: archives spec, marks task done in `tasks.md`, tags commit.
4. On rejection: feeds back into `run-loop.sh` with human notes.

---

## What the harness contains

```
harness/
  guides/         ← what the agent reads before coding
    AGENTS.md
    architecture.md
    coding-rules.md
  sensors/        ← automated validation (lint, tests, structural checks)
    check.sh
  loops/          ← loop state per feature
    tasks.md
    progress.md
    task-template.md
specs/
  product/        ← vision, data model, non-goals
  features/       ← one file per task, includes BDD acceptance criteria
```

Coding style rules live in `guides/coding-rules.md` and are file-type specific:
- `.md` → clean, concise, no bloat
- `.jsx` → clean separation of elements
- `.test.js` → one assert per test, prioritize coverage

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

These come after the six-script loop works end to end.
