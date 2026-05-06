# Plugin Installations

## Overview

Topic file tracking project-scope plugin installations via the `claude plugin` CLI. Project-scope plugins are recorded in `.claude/settings.json` (committed to git, so the project travels with its plugin set). User-scope installations are NOT tracked here — they're per-machine and not part of the repo.

## Open Questions

- Should the three already-on-disk obsidian skills (`obsidian-vault-workflow`, `obsidian-bases`, `obsidian-markdown` under `.claude/skills/`) be migrated to plugin-managed installs (registering them via `claude plugin marketplace add` from a personal marketplace) instead of staying as untracked loose files? This came up during this session but wasn't acted on.

## Session Log

### 2026-05-06 — install skill-creator at project scope [shipped]

- **What was done:** Ran `claude plugin install skill-creator@claude-plugins-official --scope project`. Step 1 of the user's fallback chain succeeded on first try — no need for steps 2 (marketplace add) or 3 (alternate marketplace). The install created `.claude/settings.json` with `enabledPlugins: { "skill-creator@claude-plugins-official": true }`. Verified via `claude plugin list`: skill-creator now appears at both `user` (pre-existing) and `project` (this install) scopes.
- **Decisions:**
  - Tracked the topic as `plugin-installations` (plural, future-proof) rather than `skill-creator-install` (single-event). Future project-scope plugin installs will append session-log entries here.
  - `.claude/settings.json` is committed — that's the whole point of `--scope project`: the plugin set travels with the repo.
  - Did NOT touch the user-scope install of skill-creator; both scopes coexist.
- **Notes / Caveats:**
  - The `claude` CLI WAS available on PATH (`/c/Users/zerem/AppData/Roaming/npm/claude`, version 2.1.118). The earlier session-wide instruction to avoid CLI tools was overridden for this specific task.
  - Source skill: https://github.com/anthropics/skills/tree/main/skills/skill-creator (per the user's request); installed via the `claude-plugins-official` marketplace, which already had it.
- **Related:** [[obsidian-vault-workflow-bootstrap]] — the prior session about getting the vault workflow active.
