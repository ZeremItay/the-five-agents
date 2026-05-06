# yuval ↔ gpt-image-gen Skill

This is a **pointer-doc for humans**, not a Claude Code skill definition.

The canonical skill definition lives at:

> [`.claude/skills/gpt-image-gen/SKILL.md`](../.claude/skills/gpt-image-gen/SKILL.md)

That's the file Claude Code loads when an agent invokes the `gpt-image-gen` skill. **Do not** edit this `skill.md` expecting it to change anything — Claude Code does not read it. Edit the SKILL.md under `.claude/skills/` instead.

## Who calls the skill

- **`yuval`** is the primary caller. Yuval scans `reference/`, synthesizes a prompt, and asks `gpt-image-gen` to render it.
- Other agents may call `gpt-image-gen` directly if they don't need yuval's style-anchoring behavior — but the convention is **route through yuval** for any visual produced by the project, so style stays consistent.

## What the skill does

A single Bash recipe that:
1. Loads `OPENAI_API_KEY` from `.env`.
2. Calls `POST https://api.openai.com/v1/images/generations` (model: `gpt-image-2`).
3. Decodes the base64 response into a PNG on disk (with `jq + base64`, falling back to `python` for environments where `jq` isn't installed — e.g. Git Bash on Windows).
4. Verifies the file exists and is non-empty before returning success.

See [`.claude/skills/gpt-image-gen/SKILL.md`](../.claude/skills/gpt-image-gen/SKILL.md) for the full recipe and error-handling table.

## Prerequisites

- `OPENAI_API_KEY` must be set in `.env`. See `.env.example` for the canonical template.
- `python` or `jq` must be on PATH (the skill uses whichever is available to decode the base64 response).
