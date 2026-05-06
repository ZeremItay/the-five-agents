# Design: `obsidian-vault-workflow` skill + initial vault

**Date:** 2026-05-06
**Status:** Approved (verbal, in auto mode)
**Related repo:** [the-five-agents](https://github.com/ZeremItay/the-five-agents)

## Context

"The Five Agents" is a content-creation agent team (CEO + sub-agents, to be defined incrementally). The user wants a single skill — `obsidian-vault-workflow` — that:

1. Scans every project file and produces a per-file Markdown doc (purpose, owner, related files).
2. Outputs the docs as an Obsidian-native vault (YAML frontmatter, `[[wiki-links]]`, `#tags`) so the user can browse the project's architecture inside Obsidian.
3. Auto-activates whenever the project structure changes or a new session starts, so the vault stays in sync without the user asking.

The skill must live inside the project's `.claude/skills/` so it travels with the repo.

## Audience & format

Two readers, **primary = the user** in Obsidian:

- Obsidian-native syntax: YAML frontmatter, `[[wiki-links]]`, `#tag` notation.
- Future Claude sessions read the same files as plain Markdown — wiki-links degrade gracefully to readable text.
- Tradeoff accepted: GitHub renders `[[]]` as plain text (not clickable). The vault is for browsing in Obsidian; GitHub is just storage.

## Vault location & layout

Root: **`docs/vault/`** (separate from `.claude/` which is runtime config, and from `docs/superpowers/specs/` which is this skill's spec home).

```
docs/vault/
├── MOC.md                          ← Map of Contents — primary entry point
├── agents/
│   └── _index.md                   ← placeholder, will fill as agents are defined
├── commands/
│   └── _index.md                   ← placeholder
├── config/
│   ├── CLAUDE-md.md                ← doc for /CLAUDE.md
│   ├── env.md                      ← doc covering .env + .env.example
│   └── gitignore.md                ← doc for /.gitignore
└── skills/
    ├── _index.md                   ← overview of all installed skills
    └── superpowers/
        ├── brainstorming.md
        ├── dispatching-parallel-agents.md
        ├── executing-plans.md
        ├── finishing-a-development-branch.md
        ├── receiving-code-review.md
        ├── requesting-code-review.md
        ├── subagent-driven-development.md
        ├── systematic-debugging.md
        ├── test-driven-development.md
        ├── using-git-worktrees.md
        ├── using-superpowers.md
        ├── verification-before-completion.md
        ├── writing-plans.md
        └── writing-skills.md
```

## Per-file doc template

Every doc under `docs/vault/` follows this structure:

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

Rules:
- `owner` for skills installed from a third-party plugin = `external`.
- `owner` for project-infra files (CLAUDE.md, .gitignore, .env*) = `project-infra`.
- `owner` for agents = the agent's own name; for the CEO = `CEO`.
- `related` always uses wiki-link form, no extension.
- File names use kebab-case; dots in source filenames become hyphens (`CLAUDE.md` → `CLAUDE-md.md`).

## Scope of the first pass

**Documented now:**
- `/CLAUDE.md` (project overview)
- `/.env` + `/.env.example` (combined into one `env.md`)
- `/.gitignore`
- All 14 Superpowers skills (one short summary each — does NOT duplicate the upstream `SKILL.md`, only summarizes purpose + links to it)
- The `obsidian-vault-workflow` skill itself

**Not documented yet:**
- `.gitkeep` files (trivial, no content)
- Sub-files of skills (`scripts/`, `references/`, helper md files inside each skill folder)
- The repo's own `.git/` internals
- This spec file (it lives in `docs/superpowers/specs/`, separate from the vault)

**Placeholders (created but mostly empty):**
- `agents/_index.md` — "no agents defined yet"
- `commands/_index.md` — "no custom commands yet"

## The skill: `.claude/skills/obsidian-vault-workflow/SKILL.md`

Structure:

1. **Frontmatter** — `name`, `description` (with trigger phrases like "create/modify/discuss any project file"), `when_to_use`.
2. **Goal** — keep `docs/vault/` in sync with the project, in Obsidian-native format.
3. **Workflow** — three steps: (a) scan project, (b) diff against existing vault, (c) write/update missing docs and refresh `MOC.md`.
4. **Per-file doc template** — the same template shown above, embedded as the canonical reference.
5. **Decision rules** — how to choose `owner`, when to add to `related`, naming conventions.
6. **Escape hatch** — when to STOP and ask the user (e.g. ambiguous owner, file looks like secrets).

## Auto-invocation

Two-layer approach:

1. **Skill description trigger** — the skill's `description` field includes phrases like "use whenever a project file is created, modified, renamed, or discussed". Claude's normal skill-routing picks it up without an explicit invocation.

2. **SessionStart hook** in `.claude/settings.json`:
   ```json
   {
     "hooks": {
       "SessionStart": [
         {
           "hooks": [
             {
               "type": "command",
               "command": "echo 'Reminder: this project uses the obsidian-vault-workflow skill. Invoke it before answering when project files are created, modified, or discussed.'"
             }
           ]
         }
       ]
     }
   }
   ```
   This injects a one-line reminder into every new session's context so Claude consistently routes through the skill.

**Not adopting**: a `UserPromptSubmit` hook that fires per-prompt. It would invoke the skill on every "yes" / "ok" / unrelated question — noise without value. The two layers above achieve the same goal cleanly.

## Verification

After execution:
1. `tree docs/vault/` matches the layout shown above (run via `find docs/vault -print` since `tree` may not be installed).
2. `docs/vault/MOC.md` exists and links to each top-level subdirectory.
3. Each documented file's doc parses as valid YAML frontmatter (manual visual check is sufficient — no parser needed for v1).
4. `.claude/skills/obsidian-vault-workflow/SKILL.md` exists with the structure described above.
5. `.claude/settings.json` exists with the SessionStart hook above; opening a new Claude Code session prints the reminder.
6. `git log --oneline -5` shows the spec commit and the implementation commit.
7. `git status` is clean and `git push` succeeds against `origin/main`.

## Out of scope (explicit non-goals)

- Documenting every sub-file inside the Superpowers skills (`scripts/`, `references/`, etc.). The summary at skill level is enough; sub-files are upstream's concern.
- Auto-detecting renames or deletions in the vault. v1 is additive: the workflow adds new docs, but doesn't garbage-collect stale ones. We'll add that when it bites.
- Building a graph view, dataview queries, or any Obsidian-plugin-specific features. Stick to portable Markdown + frontmatter.
- A `UserPromptSubmit` hook (per the auto-invocation section).
