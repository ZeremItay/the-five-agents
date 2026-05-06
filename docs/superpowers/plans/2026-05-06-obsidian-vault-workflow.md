# Obsidian Vault Workflow — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Install a project-local skill (`obsidian-vault-workflow`) that documents every project file as an Obsidian-style Markdown vault, plus seed the initial vault and wire a SessionStart hook so the workflow auto-engages.

**Architecture:** A single skill at `.claude/skills/obsidian-vault-workflow/SKILL.md` defines the workflow (scan → diff → write/update → refresh MOC). A SessionStart hook in `.claude/settings.json` injects a one-line reminder so Claude routes through the skill. The vault lives at `docs/vault/` with category subdirectories (`agents/`, `commands/`, `config/`, `skills/`) and a top-level `MOC.md`.

**Tech Stack:** Markdown + YAML frontmatter, Obsidian wiki-link conventions, Claude Code hooks.

**Adaptation note:** This plan creates static content (Markdown + JSON), not code. The standard TDD red→green flow doesn't apply, so each task uses **Write → Verify → Commit** instead. "Verify" steps confirm the file landed where intended with the right structure.

---

## Spec reference

Source spec: `docs/superpowers/specs/2026-05-06-obsidian-vault-workflow-design.md`. Read it before starting Task 1.

## File map

Files this plan creates (none modified):

```
docs/superpowers/plans/2026-05-06-obsidian-vault-workflow.md  ← this file
.claude/skills/obsidian-vault-workflow/SKILL.md               ← the skill
.claude/settings.json                                          ← SessionStart hook
docs/vault/MOC.md                                              ← Map of Contents
docs/vault/agents/_index.md                                    ← placeholder index
docs/vault/commands/_index.md                                  ← placeholder index
docs/vault/skills/_index.md                                    ← skills overview
docs/vault/config/CLAUDE-md.md                                 ← doc for CLAUDE.md
docs/vault/config/env.md                                       ← doc for .env + .env.example
docs/vault/config/gitignore.md                                 ← doc for .gitignore
docs/vault/skills/superpowers/brainstorming.md
docs/vault/skills/superpowers/dispatching-parallel-agents.md
docs/vault/skills/superpowers/executing-plans.md
docs/vault/skills/superpowers/finishing-a-development-branch.md
docs/vault/skills/superpowers/receiving-code-review.md
docs/vault/skills/superpowers/requesting-code-review.md
docs/vault/skills/superpowers/subagent-driven-development.md
docs/vault/skills/superpowers/systematic-debugging.md
docs/vault/skills/superpowers/test-driven-development.md
docs/vault/skills/superpowers/using-git-worktrees.md
docs/vault/skills/superpowers/using-superpowers.md
docs/vault/skills/superpowers/verification-before-completion.md
docs/vault/skills/superpowers/writing-plans.md
docs/vault/skills/superpowers/writing-skills.md
```

Total: 25 new files. No existing files are modified.

## Commit strategy

Three commits:
1. **Commit A:** Spec + plan (the design artifacts).
2. **Commit B:** Skill definition + SessionStart hook (the workflow infrastructure).
3. **Commit C:** All vault content (MOC + 23 docs).

Push to `origin/main` after Commit C.

---

## Task 1: Commit the design artifacts (spec + plan)

**Files:**
- Already on disk: `docs/superpowers/specs/2026-05-06-obsidian-vault-workflow-design.md`
- Already on disk: `docs/superpowers/plans/2026-05-06-obsidian-vault-workflow.md` (this file)

- [ ] **Step 1: Verify both files exist**

Run: `ls -la docs/superpowers/specs/ docs/superpowers/plans/`
Expected: Both directories list the dated files above, non-zero size.

- [ ] **Step 2: Stage both files**

Run: `git add docs/superpowers/specs/2026-05-06-obsidian-vault-workflow-design.md docs/superpowers/plans/2026-05-06-obsidian-vault-workflow.md`

- [ ] **Step 3: Commit**

```bash
git -c user.email="zeremitay.mcc@gmail.com" -c user.name="ZeremItay" commit -m "$(cat <<'EOF'
Add obsidian-vault-workflow spec and implementation plan

Design and step-by-step plan for a project-local skill that
keeps an Obsidian-style vault under docs/vault/ in sync with
project files, plus a SessionStart hook for auto-routing.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

Expected: New commit created on `main`. Note the hash for the summary.

---

## Task 2: Create the skill file

**Files:**
- Create: `.claude/skills/obsidian-vault-workflow/SKILL.md`

- [ ] **Step 1: Write `.claude/skills/obsidian-vault-workflow/SKILL.md`**

Use the Write tool. Full content:

````markdown
---
name: obsidian-vault-workflow
description: Use whenever a project file is created, modified, renamed, or discussed — this skill keeps the Obsidian-style vault under docs/vault/ in sync. Also use at the start of every session to confirm the vault matches reality.
when_to_use: Project file lifecycle events (create / modify / rename / delete / discuss), session starts, or whenever the user asks about project structure.
---

# Obsidian Vault Workflow

Keep `docs/vault/` in sync with the project. Every meaningful project file gets a Markdown doc with YAML frontmatter, written in Obsidian-native style (wiki-links `[[]]`, `#tags`).

## When to invoke

- A project file was created, renamed, deleted, or substantively changed.
- A session is starting and you have not yet checked the vault for drift.
- The user asks "what is X" / "where does Y live" / "who owns Z" about a project file.

## Workflow

### 1. Scan
Enumerate project files worth documenting:
- All non-hidden files at the repo root (`CLAUDE.md`, `.env.example`, `.gitignore`, etc. — note hidden files starting with `.` ARE in scope when they're config).
- All agent definitions under `.claude/agents/`.
- All custom commands under `.claude/commands/`.
- All skills under `.claude/skills/` (one summary per skill folder, NOT per inner file).
- Any source code added later (use judgment: a single helper module = one doc; deeply nested internals are out of scope).

Out of scope: `.git/`, `.gitkeep`, `node_modules/`, build artifacts, anything matched by `.gitignore`.

### 2. Diff
Compare the scan against `docs/vault/`. For each scanned file:
- If a corresponding doc exists, check whether content changed enough to warrant updating the summary or `related` links.
- If no doc exists, mark for creation.
- If a doc exists but the source file no longer exists, flag it for the user — do NOT auto-delete.

### 3. Write / update
For each missing or stale doc:
- Use the template below.
- File name = source file name with dots → hyphens, kebab-case (`CLAUDE.md` → `CLAUDE-md.md`, `.gitignore` → `gitignore.md`).
- Place under the right category subfolder (`agents/`, `commands/`, `config/`, `skills/...`).
- Add an entry under the correct heading in `MOC.md`.
- Refresh `related` links in any doc that mentions the new file.

## Per-file doc template

```markdown
---
file: <relative path from repo root, or "n/a" for index/MOC>
type: agent | skill | command | config | doc | index
owner: CEO | <agent-name> | project-infra | external
status: active | draft | deprecated | placeholder
tags: [tag1, tag2]
related:
  - "[[other-doc-1]]"
  - "[[other-doc-2]]"
---

# <Descriptive title>

**What it does:** one or two sentences.

**Belongs to:** <owner — same value as frontmatter, repeated for human readability>.

**Related:**
- [[wiki-link-1]] — why it's relevant
- [[wiki-link-2]] — ...

## Notes
<optional longer prose; omit if nothing to add>
```

## Decision rules

**`owner`:**
- Files installed from a third-party plugin (e.g. anything under `.claude/skills/<plugin>/`) → `external`.
- Project-infra files (`CLAUDE.md`, `.gitignore`, `.env*`, `settings.json`, this skill itself) → `project-infra`.
- An agent definition → the agent's own name (e.g. `editor`, `researcher`).
- The orchestrator agent → `CEO`.

**`related`:** Add a wiki-link when one file directly references, depends on, or modifies the behavior of another. Don't add links for vague thematic association — keep the graph honest.

**Naming:** kebab-case. Replace dots with hyphens (`CLAUDE.md` → `CLAUDE-md.md`). Drop file extensions from wiki-links: `[[CLAUDE-md]]`, not `[[CLAUDE-md.md]]`.

## Stop and ask

Stop and ask the user when:
- A file's `owner` is genuinely unclear (e.g. a script that several agents share).
- A source file appears to contain secrets (don't write it into the vault).
- A doc exists but its source file is gone — could be a rename or a deletion; user decides.

## Out of scope (v1)

- Auto-detecting renames / deletions (manual prompt instead).
- Sub-files inside skills (`scripts/`, `references/`) — only top-level skill summary.
- Dataview / graph view / other Obsidian-plugin-specific features.
- A `UserPromptSubmit` hook (too noisy — see settings.json comment).
````

- [ ] **Step 2: Verify file exists with frontmatter**

Run: `head -5 .claude/skills/obsidian-vault-workflow/SKILL.md`
Expected: Output begins with `---` and shows `name: obsidian-vault-workflow`.

---

## Task 3: Create the SessionStart hook

**Files:**
- Create: `.claude/settings.json`

- [ ] **Step 1: Write `.claude/settings.json`**

Full content:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo Reminder: this project uses the obsidian-vault-workflow skill. Invoke it whenever project files are created, modified, renamed, or discussed."
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 2: Verify JSON is valid**

Run: `python -c "import json; json.load(open('.claude/settings.json')); print('OK')"`
Expected: Prints `OK`. (If `python` is missing, use Node: `node -e "JSON.parse(require('fs').readFileSync('.claude/settings.json'))"`.)

- [ ] **Step 3: Commit Task 2 + Task 3 together**

```bash
git add .claude/skills/obsidian-vault-workflow/SKILL.md .claude/settings.json
git -c user.email="zeremitay.mcc@gmail.com" -c user.name="ZeremItay" commit -m "$(cat <<'EOF'
Add obsidian-vault-workflow skill and SessionStart hook

The skill defines a Scan → Diff → Write/Update workflow that
keeps docs/vault/ in sync with project files. The SessionStart
hook injects a reminder so Claude routes through the skill on
every new session without an explicit invocation.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

Expected: New commit on `main`. Note the hash.

---

## Task 4: Create the vault skeleton (MOC + placeholder indexes)

**Files:**
- Create: `docs/vault/MOC.md`
- Create: `docs/vault/agents/_index.md`
- Create: `docs/vault/commands/_index.md`
- Create: `docs/vault/skills/_index.md`

- [ ] **Step 1: Write `docs/vault/MOC.md`**

```markdown
---
file: n/a
type: index
owner: project-infra
status: active
tags: [moc, vault-root]
related: []
---

# Map of Contents

Top-level entry point for the vault. Every documented file links from here, directly or via a category index.

## Categories

- [[agents/_index|Agents]] — the CEO and sub-agents.
- [[commands/_index|Commands]] — custom slash commands for this project.
- [[skills/_index|Skills]] — installed skills (Superpowers + project-local).
- **Config** — project-infra files:
  - [[config/CLAUDE-md|CLAUDE.md]] — top-level project guidance.
  - [[config/env|.env / .env.example]] — environment variables.
  - [[config/gitignore|.gitignore]] — what stays out of git.

## Conventions

- File names use kebab-case; dots in source filenames become hyphens.
- Wiki-links omit the `.md` extension.
- Frontmatter is required on every doc — see `[[../skills/obsidian-vault-workflow]]` (or its source at `.claude/skills/obsidian-vault-workflow/SKILL.md`) for the canonical template.
```

- [ ] **Step 2: Write `docs/vault/agents/_index.md`**

```markdown
---
file: n/a
type: index
owner: project-infra
status: placeholder
tags: [agents, index]
related:
  - "[[../MOC]]"
---

# Agents

No agents defined yet. The CEO and sub-agents will be added under `.claude/agents/` and documented here as they come online.

## Planned

- CEO (orchestrator) — to be defined.
- Specialist sub-agents — to be defined.
```

- [ ] **Step 3: Write `docs/vault/commands/_index.md`**

```markdown
---
file: n/a
type: index
owner: project-infra
status: placeholder
tags: [commands, index]
related:
  - "[[../MOC]]"
---

# Commands

No custom slash commands defined yet. Project-specific commands will live under `.claude/commands/` and be documented here.
```

- [ ] **Step 4: Write `docs/vault/skills/_index.md`**

```markdown
---
file: n/a
type: index
owner: project-infra
status: active
tags: [skills, index]
related:
  - "[[../MOC]]"
  - "[[../../skills/obsidian-vault-workflow|obsidian-vault-workflow source]]"
---

# Skills

Two sources:

1. **Superpowers** (external, MIT — Jesse Vincent / [obra/superpowers](https://github.com/obra/superpowers) v5.1.0). Installed under `.claude/skills/`. Summarized in [[superpowers/_overview|the superpowers folder]] below.
2. **Project-local** — built specifically for the five-agents project.

## Superpowers (14)

- [[superpowers/brainstorming]]
- [[superpowers/dispatching-parallel-agents]]
- [[superpowers/executing-plans]]
- [[superpowers/finishing-a-development-branch]]
- [[superpowers/receiving-code-review]]
- [[superpowers/requesting-code-review]]
- [[superpowers/subagent-driven-development]]
- [[superpowers/systematic-debugging]]
- [[superpowers/test-driven-development]]
- [[superpowers/using-git-worktrees]]
- [[superpowers/using-superpowers]]
- [[superpowers/verification-before-completion]]
- [[superpowers/writing-plans]]
- [[superpowers/writing-skills]]

## Project-local

- **obsidian-vault-workflow** — keeps this vault in sync. Source: `.claude/skills/obsidian-vault-workflow/SKILL.md`.
```

- [ ] **Step 5: Verify all four files exist**

Run: `ls docs/vault/ docs/vault/agents/ docs/vault/commands/ docs/vault/skills/`
Expected: `MOC.md` at vault root; `_index.md` in each subdirectory.

---

## Task 5: Create config docs

**Files:**
- Create: `docs/vault/config/CLAUDE-md.md`
- Create: `docs/vault/config/env.md`
- Create: `docs/vault/config/gitignore.md`

- [ ] **Step 1: Write `docs/vault/config/CLAUDE-md.md`**

```markdown
---
file: CLAUDE.md
type: config
owner: project-infra
status: active
tags: [project-overview, claude-code]
related:
  - "[[../MOC]]"
  - "[[gitignore]]"
  - "[[env]]"
  - "[[../skills/_index|Skills index]]"
---

# CLAUDE.md

**What it does:** Top-level guidance loaded into every Claude Code session in this repo. Names the project, describes the agent-team concept (CEO + specialists), and points at the `.claude/` layout.

**Belongs to:** project-infra.

**Related:**
- [[../MOC]] — vault entry point.
- [[../skills/_index]] — what lives under `.claude/skills/`.
- [[gitignore]] — companion config.
- [[env]] — companion config.

## Notes

This file grows as the team takes shape. Keep it short — it's the elevator pitch for future Claude sessions, not exhaustive documentation.
```

- [ ] **Step 2: Write `docs/vault/config/env.md`**

```markdown
---
file: .env, .env.example
type: config
owner: project-infra
status: active
tags: [secrets, environment]
related:
  - "[[../MOC]]"
  - "[[gitignore]]"
  - "[[CLAUDE-md]]"
---

# .env / .env.example

**What it does:** `.env.example` is the canonical, committed template of environment variables. `.env` is the local-only file with real values; it is git-ignored. Currently the only variable defined is `ANTHROPIC_API_KEY`.

**Belongs to:** project-infra.

**Related:**
- [[gitignore]] — defines that `.env` is excluded.
- [[CLAUDE-md]] — top-level project guidance.

## Notes

Convention: any new variable goes into BOTH `.env.example` (with empty value) and `.env` (with the real value). Keep them structurally identical so collaborators can copy `.env.example` → `.env` and just fill in values.
```

- [ ] **Step 3: Write `docs/vault/config/gitignore.md`**

```markdown
---
file: .gitignore
type: config
owner: project-infra
status: active
tags: [git, hygiene]
related:
  - "[[../MOC]]"
  - "[[env]]"
---

# .gitignore

**What it does:** Excludes secrets (`.env*` except `.env.example`), OS noise (`.DS_Store`, `Thumbs.db`, `desktop.ini`), editor caches (`.vscode/`, `.idea/`), Python and Node build artifacts, and Claude Code's local-only `settings.local.json`.

**Belongs to:** project-infra.

**Related:**
- [[env]] — the file this primarily protects.

## Notes

Add new ignores by category (Python / Node / IDE / etc.) rather than by single file when possible — it stays readable as the project grows.
```

- [ ] **Step 4: Verify all three files exist**

Run: `ls docs/vault/config/`
Expected: `CLAUDE-md.md  env.md  gitignore.md`.

---

## Task 6: Create the 14 Superpowers skill summaries

**Files (all under `docs/vault/skills/superpowers/`):** 14 files, listed in the file map at the top of this plan.

Each file follows the same template. The only varying parts are: `<skill-name>`, the one-line "what it does" summary, and the `tags`. The `owner` is always `external`, `status` always `active`, and the `related` list always points to `[[_index]]` at minimum.

- [ ] **Step 1: Write `docs/vault/skills/superpowers/brainstorming.md`**

```markdown
---
file: .claude/skills/brainstorming/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, design, planning]
related:
  - "[[_index|Skills index]]"
  - "[[writing-plans]]"
  - "[[writing-skills]]"
---

# brainstorming

**What it does:** Turns a fuzzy idea into an approved design via one-question-at-a-time dialogue, then writes a spec under `docs/superpowers/specs/`. Required before any creative work.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[writing-plans]] — the next step after a design is approved.
- [[writing-skills]] — used when the thing being designed is itself a skill.

## Notes

Source: `.claude/skills/brainstorming/SKILL.md`. The terminal state is invoking writing-plans — never another implementation skill directly.
```

- [ ] **Step 2: Write `docs/vault/skills/superpowers/dispatching-parallel-agents.md`**

```markdown
---
file: .claude/skills/dispatching-parallel-agents/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, agents, parallelism]
related:
  - "[[_index|Skills index]]"
  - "[[subagent-driven-development]]"
---

# dispatching-parallel-agents

**What it does:** Pattern for sending multiple Task-tool subagents in parallel when 2+ subtasks are independent. Reduces wall-clock time and keeps the main context clean.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[subagent-driven-development]] — the broader workflow this fits inside.

## Notes

Source: `.claude/skills/dispatching-parallel-agents/SKILL.md`.
```

- [ ] **Step 3: Write `docs/vault/skills/superpowers/executing-plans.md`**

```markdown
---
file: .claude/skills/executing-plans/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, execution]
related:
  - "[[_index|Skills index]]"
  - "[[writing-plans]]"
  - "[[subagent-driven-development]]"
  - "[[verification-before-completion]]"
---

# executing-plans

**What it does:** Runs a written implementation plan inline (in the current session) with checkpoints between batches of tasks. Alternative to subagent-driven-development.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[writing-plans]] — produces the plan this consumes.
- [[subagent-driven-development]] — the alternative execution mode.
- [[verification-before-completion]] — required before claiming done.

## Notes

Source: `.claude/skills/executing-plans/SKILL.md`.
```

- [ ] **Step 4: Write `docs/vault/skills/superpowers/finishing-a-development-branch.md`**

```markdown
---
file: .claude/skills/finishing-a-development-branch/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, git, completion]
related:
  - "[[_index|Skills index]]"
  - "[[verification-before-completion]]"
  - "[[requesting-code-review]]"
---

# finishing-a-development-branch

**What it does:** Decision guide for completing a branch — merge, open a PR, or just clean up. Walks through verification, review options, and integration paths.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[verification-before-completion]] — must pass first.
- [[requesting-code-review]] — invoked when a PR/review is the right path.

## Notes

Source: `.claude/skills/finishing-a-development-branch/SKILL.md`.
```

- [ ] **Step 5: Write `docs/vault/skills/superpowers/receiving-code-review.md`**

```markdown
---
file: .claude/skills/receiving-code-review/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, review]
related:
  - "[[_index|Skills index]]"
  - "[[requesting-code-review]]"
---

# receiving-code-review

**What it does:** How to handle code-review feedback rigorously — verify each suggestion, push back when it's wrong, avoid blind agreement and avoid blind dismissal.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[requesting-code-review]] — the symmetric counterpart.

## Notes

Source: `.claude/skills/receiving-code-review/SKILL.md`.
```

- [ ] **Step 6: Write `docs/vault/skills/superpowers/requesting-code-review.md`**

```markdown
---
file: .claude/skills/requesting-code-review/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, review]
related:
  - "[[_index|Skills index]]"
  - "[[receiving-code-review]]"
  - "[[finishing-a-development-branch]]"
---

# requesting-code-review

**What it does:** Triggers a structured code review against the original requirements and project standards after a meaningful chunk of work is done.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[receiving-code-review]] — companion skill for the reviewee side.
- [[finishing-a-development-branch]] — usually called from there.

## Notes

Source: `.claude/skills/requesting-code-review/SKILL.md`. Bundles a `code-reviewer.md` reviewer prompt.
```

- [ ] **Step 7: Write `docs/vault/skills/superpowers/subagent-driven-development.md`**

```markdown
---
file: .claude/skills/subagent-driven-development/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, execution, agents]
related:
  - "[[_index|Skills index]]"
  - "[[writing-plans]]"
  - "[[executing-plans]]"
  - "[[dispatching-parallel-agents]]"
---

# subagent-driven-development

**What it does:** Executes a plan one task at a time by dispatching a fresh subagent per task, with two-stage review between tasks. Recommended over inline execution for non-trivial plans.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[writing-plans]] — produces the plan this consumes.
- [[executing-plans]] — the inline-execution alternative.
- [[dispatching-parallel-agents]] — used when independent tasks can run together.

## Notes

Source: `.claude/skills/subagent-driven-development/SKILL.md`. Bundles `implementer-prompt.md`, `spec-reviewer-prompt.md`, and `code-quality-reviewer-prompt.md`.
```

- [ ] **Step 8: Write `docs/vault/skills/superpowers/systematic-debugging.md`**

```markdown
---
file: .claude/skills/systematic-debugging/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, debugging]
related:
  - "[[_index|Skills index]]"
  - "[[verification-before-completion]]"
  - "[[test-driven-development]]"
---

# systematic-debugging

**What it does:** Disciplined approach to bugs and unexpected behavior — reproduce, narrow, hypothesize, verify, root-cause. Required before proposing any fix.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[verification-before-completion]] — used to confirm the fix actually fixed it.
- [[test-driven-development]] — write the failing regression test first.

## Notes

Source: `.claude/skills/systematic-debugging/SKILL.md`. Includes references on root-cause tracing, condition-based waiting, and defense in depth.
```

- [ ] **Step 9: Write `docs/vault/skills/superpowers/test-driven-development.md`**

```markdown
---
file: .claude/skills/test-driven-development/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, testing, tdd]
related:
  - "[[_index|Skills index]]"
  - "[[systematic-debugging]]"
  - "[[verification-before-completion]]"
---

# test-driven-development

**What it does:** Red → green → refactor discipline. Required before writing any feature or bug-fix code.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[systematic-debugging]] — uses TDD for the regression test.
- [[verification-before-completion]] — TDD doesn't replace final verification.

## Notes

Source: `.claude/skills/test-driven-development/SKILL.md`. Bundles `testing-anti-patterns.md`.
```

- [ ] **Step 10: Write `docs/vault/skills/superpowers/using-git-worktrees.md`**

```markdown
---
file: .claude/skills/using-git-worktrees/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, git, isolation]
related:
  - "[[_index|Skills index]]"
  - "[[finishing-a-development-branch]]"
---

# using-git-worktrees

**What it does:** Creates isolated worktrees for feature work or plan execution, with directory-selection and safety-verification helpers.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[finishing-a-development-branch]] — common downstream step.

## Notes

Source: `.claude/skills/using-git-worktrees/SKILL.md`.
```

- [ ] **Step 11: Write `docs/vault/skills/superpowers/using-superpowers.md`**

```markdown
---
file: .claude/skills/using-superpowers/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, meta]
related:
  - "[[_index|Skills index]]"
  - "[[brainstorming]]"
---

# using-superpowers

**What it does:** Meta-skill — describes how to discover, invoke, and prioritize the rest of the Superpowers skills. Loaded at session start.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[brainstorming]] — the most common first invocation.

## Notes

Source: `.claude/skills/using-superpowers/SKILL.md`. References include tool-name mappings for Codex, Copilot CLI, and Gemini CLI.
```

- [ ] **Step 12: Write `docs/vault/skills/superpowers/verification-before-completion.md`**

```markdown
---
file: .claude/skills/verification-before-completion/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, completion]
related:
  - "[[_index|Skills index]]"
  - "[[finishing-a-development-branch]]"
  - "[[systematic-debugging]]"
---

# verification-before-completion

**What it does:** Gate before claiming any work is done. Run the verifying commands, confirm the output, then make the success claim — never the other way round.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[finishing-a-development-branch]] — calls this as a prerequisite.
- [[systematic-debugging]] — calls this to confirm a fix.

## Notes

Source: `.claude/skills/verification-before-completion/SKILL.md`.
```

- [ ] **Step 13: Write `docs/vault/skills/superpowers/writing-plans.md`**

```markdown
---
file: .claude/skills/writing-plans/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, planning]
related:
  - "[[_index|Skills index]]"
  - "[[brainstorming]]"
  - "[[executing-plans]]"
  - "[[subagent-driven-development]]"
---

# writing-plans

**What it does:** Turns an approved spec into a bite-sized, exact-paths, no-placeholders implementation plan saved under `docs/superpowers/plans/`.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[brainstorming]] — produces the spec this consumes.
- [[executing-plans]] / [[subagent-driven-development]] — execute the plan this produces.

## Notes

Source: `.claude/skills/writing-plans/SKILL.md`. Bundles a `plan-document-reviewer-prompt.md`.
```

- [ ] **Step 14: Write `docs/vault/skills/superpowers/writing-skills.md`**

```markdown
---
file: .claude/skills/writing-skills/SKILL.md
type: skill
owner: external
status: active
tags: [superpowers, meta, authoring]
related:
  - "[[_index|Skills index]]"
  - "[[brainstorming]]"
---

# writing-skills

**What it does:** Guide for creating new skills, editing existing ones, and verifying they work before deployment.

**Belongs to:** external (Superpowers, MIT).

**Related:**
- [[brainstorming]] — design the skill first.

## Notes

Source: `.claude/skills/writing-skills/SKILL.md`. Bundles references on persuasion, anthropic best practices, graphviz conventions, and an example test plan.
```

- [ ] **Step 15: Verify all 14 files exist**

Run: `ls docs/vault/skills/superpowers/ | wc -l`
Expected: `14`.

---

## Task 7: Final stage — vault commit + push

**Files:** all under `docs/vault/`.

- [ ] **Step 1: Stage all vault files**

Run: `git add docs/vault/`

- [ ] **Step 2: Confirm what's staged**

Run: `git status --short`
Expected: 23 new files under `docs/vault/`, nothing else.

- [ ] **Step 3: Commit**

```bash
git -c user.email="zeremitay.mcc@gmail.com" -c user.name="ZeremItay" commit -m "$(cat <<'EOF'
Seed docs/vault/ with the initial Obsidian-style vault

Adds the MOC, category indexes (agents, commands, skills),
config docs (CLAUDE.md, env, gitignore), and a 14-entry
superpowers skill summary set. Each doc uses the canonical
frontmatter and wiki-link conventions defined by the
obsidian-vault-workflow skill.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 4: Push all commits**

Run: `git push origin main`
Expected: Three new commits land on `origin/main` (Tasks 1, 3, 7).

- [ ] **Step 5: Final verification**

Run: `git log --oneline -6` and `find docs/vault -type f | sort`
Expected:
- Log shows the three new commits plus the prior three (`cb881d2`, `837ddf7`, `ba70679`).
- File list shows exactly 23 files under `docs/vault/`.

---

## Self-review

**Spec coverage check:**
- Audience & format → addressed by template in Task 2 + content in Tasks 4–6. ✓
- Vault location & layout → Tasks 4–6 create the exact tree from spec section 2. ✓
- Per-file doc template → encoded in Task 2 SKILL.md and reused in Tasks 4–6. ✓
- Scope of first pass → Tasks 4 (placeholders) + 5 (config) + 6 (skills). ✓
- Skill at `.claude/skills/obsidian-vault-workflow/SKILL.md` → Task 2. ✓
- SessionStart hook → Task 3. ✓
- No `UserPromptSubmit` hook → confirmed absent in Task 3 settings.json. ✓
- Verification → Task 7 final step. ✓

**Placeholder scan:** No "TBD" / "TODO" / "implement later" / "fill in" anywhere. ✓

**Type consistency:** Naming convention (kebab-case, dots → hyphens, no `.md` in wiki-links) is applied uniformly across Tasks 2, 4, 5, 6. ✓

No issues found.
