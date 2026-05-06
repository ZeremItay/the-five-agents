# Obsidian Vault Workflow — Bootstrap

## Overview

First activation of the `obsidian-vault-workflow` skill on the `the-five-agents` repo. The skill (already present in `.claude/skills/obsidian-vault-workflow/SKILL.md`, untracked at the time of this entry) defines a topic-based protocol: every task reads the matching topic file in `vault/` before work and appends a dated session-log entry after work. This file is the first such entry. The vault lives at `vault/` (already created and Obsidian-recognized via `.obsidian/`); only `Meeting Notes/` is initialized in this session — the other folders (`Content Briefs/`, `Publishing Log/`, `Brand Guidelines/`) will be created on first use per the skill.

## Open Questions

- Should `.obsidian/` be tracked in git, partially tracked (e.g. `graph.json` / `core-plugins.json` yes, `workspace.json` no), or fully ignored? Currently fully untracked.
- Should the three obsidian-* skills (`obsidian-vault-workflow`, `obsidian-bases`, `obsidian-markdown`) be committed alongside the Superpowers skills already in git? They have been in `.claude/skills/` but untracked since at least 2026-05-05.
- Should `.claude/settings.json` add a `SessionStart` hook that prints a one-line reminder of this skill, or rely on Claude's normal skill routing via the skill's `description` field?
- Backfill policy: do prior 2026-05-06 actions (CLAUDE.md scaffold, env setup, GitHub push to ZeremItay/the-five-agents, Superpowers v5.1.0 install) get retroactive topic files, or do we only log forward from this entry?

## Session Log

### 2026-05-06 — vault first activation + design course-correction [shipped]

- **What was done:** Created `vault/Meeting Notes/` with `_index.md` and this topic file. The vault now has its first formal entry under the obsidian-vault-workflow protocol.
- **Decisions:**
  - Adopted the existing `obsidian-vault-workflow` skill verbatim (Option C from a three-way choice). Earlier in the session I had designed a competing per-file-documentation vault under `docs/vault/`, written a spec and plan, and committed them in `42e1093`. After discovering the existing skill on disk, the spec/plan commit was reverted in `bfd45e8` (non-destructive — `git reset --hard` was correctly blocked because a single-character "C" reply isn't explicit destruction authorization).
  - `vault/` is the canonical vault root, not `docs/vault/`. The existing skill already aligns with `vault/` and the `.obsidian/` config in the repo confirms it.
  - Only `Meeting Notes/` is created on this session — the protocol says folders are created "on first use", so `Content Briefs/`, `Publishing Log/`, `Brand Guidelines/` stay absent until needed.
- **Notes / Caveats:**
  - The miss happened because I jumped into `superpowers:brainstorming` without first checking whether a skill of the requested name already existed. Lesson: when the user names a skill, look for it on disk before designing one.
  - Two more obsidian skills (`obsidian-bases`, `obsidian-markdown`) are present but untracked — flagged as an Open Question above.
  - Earlier session work today (CLAUDE.md scaffold, env config, push to GitHub, Superpowers v5.1.0 install) is committed but not yet logged in the vault. See Open Questions for backfill policy.
  - The `42e1093` and `bfd45e8` revert pair is preserved in history rather than rewritten — `git reset` was blocked for safety, `git revert` was used instead.
- **Related:** none (first entry on this topic)
