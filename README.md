# AvangardeSpec - CLI Harness

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/Y8Y81MNWEN)

A shell-based CLI harness for human-validated, BDD-driven AI-assisted development.

**AI proposes. Sensors enforce. You approve.**

Every task runs through a structured loop: constitution → plan → pre-task → code → close.
The AI writes code. You hold the gate at each phase. Nothing merges without your sign-off.

---

## Why

AI coding tools are fast but unstructured. Without discipline, you end up with:
- Features that pass vibes but break contracts
- No repeatable way to catch regressions
- Commits that span three concerns
- The AI fixing the same bug in circles

Avangarde Harness gives AI sessions the same structure a disciplined engineer would impose:
BDD specs before coding, sensors as exit criteria, human approval at every gate.
Also, the harness uses CLI tools (repeatable results regardless of model), instead of installed agents/skills. This should save on tokens and also get the same results every time.

---

## Install

```bash
git clone https://github.com/your-org/avangarde-harness
cd avangarde-harness
chmod +x scripts/avangardespec

# Optional: make it globally available
ln -s "$(pwd)/scripts/avangardespec" /usr/local/bin/avangardespec
```

**Requirements:**
- Bash 3.2+ (macOS default works)
- [`gum`](https://github.com/charmbracelet/gum) — terminal UI (`brew install gum`)
- [`claude`](https://www.npmjs.com/package/@anthropic-ai/claude-code) CLI — `npm install -g @anthropic-ai/claude-code`
- `git`
- `coreutils` — timeout support, optional but recommended (`brew install coreutils`)

---

## Quickstart — new project

```bash
cd my-project          # must be a git repo

avangardespec init     # constitution wizard: vision, stack, rules
avangardespec plan     # AI proposes a phased task list, you approve

# Then for each task:
avangardespec pre-task # BDD spec + branch
avangardespec task     # AI coding loop + sensors
avangardespec close    # validate + merge
```

## Quickstart — existing project

```bash
cd my-existing-project

avangardespec adopt    # scans docs + code, builds harness around what exists
                       # marks done tasks [x], leaves open tasks [ ]

# Then pick up from the first open task:
avangardespec pre-task
avangardespec task
avangardespec close
```

---

## Commands

### `avangardespec init`

Run once per new project. Wizard flow:

1. Asks what you're building, your tech stack, and any constraints.
2. AI proposes: vision, data model, architecture rules, coding rules, branching strategy.
3. You approve or edit each section.
4. Scaffolds the harness structure and writes `CLAUDE.md`.

**Scaffolds:**
```
harness/
  guides/
    AGENTS.md          ← agent behavior rules
    architecture.md    ← architecture decisions
    coding-rules.md    ← file-type coding rules
  sensors/
    check.sh           ← sensor aggregator
  loops/
    tasks.md           ← task list
    progress.md        ← completion log
  scripts/             ← local copy of harness scripts
specs/
  product/
    vision.md
CLAUDE.md              ← harness instructions for Claude Code
```

---

### `avangardespec adopt`

Retrofits the harness onto an existing project. Useful when the project already has code.

1. Asks you to describe the project and what you think is missing.
2. Scans all markdown docs and key source files.
3. AI extracts the vision, tech stack, and data model from what it finds.
4. AI generates a task list, marking already-implemented features `[x]` and gaps `[ ]`.
5. You walk through each phase and approve or edit.
6. Scaffolds the harness without touching your existing code.

---

### `avangardespec plan`

Generates a phased task list from the project vision.

1. Reads `specs/product/vision.md`.
2. AI proposes a phased breakdown (2–4 phases).
3. You walk through each phase: accept or edit.
4. Writes the approved list to `harness/loops/tasks.md`.

**Task list format:**
```markdown
## Phase 1 — Foundation
[ ] DB schema + migrations
[ ] Environment config

## Phase 2 — Backend
[ ] API: list items
[ ] API: create item
```

---

### `avangardespec scope`

Adds a task mid-project (new feature, bug, hotfix).

1. Asks: what needs to be done? When? (now / next-phase / backlog)
2. AI assesses dependencies and flags urgency.
3. You approve the placement.
4. Task is inserted into `tasks.md` at the right position.

---

### `avangardespec pre-task`

Prepares everything before coding starts. Run once per task.

1. Finds the next `[ ]` task. You confirm or pick a different one.
2. AI generates BDD acceptance criteria (2–4 Given/When/Then scenarios). You approve.
3. AI detects any **manual setup required** (API keys, env vars, external accounts, etc.).
   - Surfaces them as a checklist you can tick off before or during coding.
   - Stored in the spec — re-verified at closure.
4. AI checks whether `check.sh` covers this task type. If not, it proposes new sensor checks.
5. Creates a git branch: `task/<slug>`.
6. Writes spec to `specs/features/<slug>.md`.

---

### `avangardespec task`

The Ralph loop. Runs until you exit.

```
loop:
  → AI codes against the BDD spec + guides
  → run harness/sensors/check.sh
  if pass  → offer commit → continue or exit to closure
  if fail  → surface error → AI fixes → re-run
  if same failure twice → stop → update guides / spec / sensors
```

The full BDD spec, architecture rules, and coding rules are injected into every AI session.

**Same failure twice** means the AI isn't learning from context. Options presented:
1. Update `harness/guides/architecture.md` to clarify the rule.
2. Update the spec to clarify expected behavior.
3. Update `harness/sensors/check.sh` if the check itself is wrong.
4. Override and continue anyway.

---

### `avangardespec close`

Closes the task after sensors pass.

1. Verifies all sensors are green — hard stop if not.
2. Shows any manual requirements (env vars, credentials, etc.) as an interactive checklist.
   - You tick off what's done. Blocks closure if anything remains unchecked.
3. AI diffs the implementation against the original BDD spec.
4. You approve: "Does the behavior match the intent?"
5. On approval:
   - Commits remaining changes.
   - Marks the task `[x]` in `tasks.md`.
   - Archives the spec to `specs/features/done/`.
   - Merges the task branch into your base branch.
6. On rejection:
   - You write rejection notes — appended to the spec.
   - Run `avangardespec task` to continue with updated context.

---

### `avangardespec sync`

Re-copies the harness scripts into `harness/scripts/` from the canonical repo source.
Use this after updating the harness to a newer version.

---

## Sensors

`harness/sensors/check.sh` is the single aggregator for all automated checks.
It starts empty and **grows with your project** as task types are encountered.

```bash
# Example check.sh after a few tasks:
run_check "Unit tests"  npm test
run_check "Lint"        npm run lint
run_check "Typecheck"   npx tsc --noEmit
run_check "API health"  curl -sf http://localhost:3000/health
```

Each `run_check` call runs with a timeout (10s) and reports `✓` or `✗`.
A non-zero exit from any check blocks the loop and surfaces the failure to the AI.

**Rule:** if the same sensor fails twice, update the guides — not just the code.

---

## Manual requirements

When `avangardespec pre-task` generates the spec, it also asks the AI to identify anything
a human must do that code cannot — provisioning `.env` variables, registering API keys,
creating cloud accounts, setting up OAuth apps, etc.

These appear as an interactive checklist at two points:
- **Before coding** (`avangardespec task`): tick off what's already done.
- **Before closing** (`avangardespec close`): all items must be checked before merge.

The harness does not let you close a task with outstanding manual requirements.

---

## Project structure (inside your project after init)

```
harness/
  .config                    ← base branch, internal settings
  guides/
    AGENTS.md
    architecture.md
    coding-rules.md
  sensors/
    check.sh
  loops/
    tasks.md
    progress.md
  scripts/                   ← local copy of all harness scripts
specs/
  product/
    vision.md
  features/
    <slug>.md                ← active spec per task
    done/
      <slug>-YYYYMMDD.md    ← archived after closure
CLAUDE.md
```

---

## Key concepts

| Concept | Role |
|---|---|
| **Constitution** | Vision, rules, and constraints agreed upfront — reloaded every session |
| **Spec** | Per-task: BDD criteria + manual requirements = definition of done |
| **Sensor** | Automated check enforcing a quality rule (test, lint, typecheck, process exit, etc.) |
| **Ralph loop** | Repeated AI coding + sensor check cycle until all sensors pass |
| **Harness** | Guides + sensors + loop state that make AI outcomes repeatable |

**Core mindset:**
- One-off mistake → fix the code.
- Repeating mistake → fix the harness (guides, spec, or sensors).

---

## Git discipline

- Every task runs on its own branch (`task/<slug>`).
- Commits are scoped: one concern, no unrelated files.
- `avangardespec task` offers a commit after each green sensor run.
- Branches merge only after `avangardespec close` human approval.
- `commit.sh` uses AI to suggest `type: description` messages.

---

## Claude integration

`scripts/claude.sh` wraps `claude --print` with a strict system prompt:
- No prose, no markdown headers.
- Structured output only: bullet lists or `key: value` pairs.
- Max ~20 lines.

This keeps AI output parseable inside wizard scripts.

`avangardespec task` launches an interactive Claude Code session with the full task context
pre-loaded (spec, architecture rules, coding rules, sensor failures if any).

---

## Next Steps
1. Installation script: Handles dependencies, adding to path, etc.
2. Code rewrite: This project was iterated only after usage in one small project. Pending testing on a bigger project, fixing bugs and reestructure the code to allow other features.
3. Fix the Claude Code dependency: Currently this script depends on claude code specifically to work. There are many providers for coding (like copilot) that have a good market share. Additionally, support for APIs should be added.
4. System prompts: this AI overrides system prompts for claude. It should be better configurable in the harness what system prompts to use.

---

## License

MIT
