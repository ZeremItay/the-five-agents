# Yuval (Creative/Visual Agent) + gpt-image-gen Skill

## Overview

Bootstrap of the project's image-generation infrastructure. Two pieces shipped together:

- **`gpt-image-gen` skill** at `.claude/skills/gpt-image-gen/SKILL.md` — shared Bash wrapper around OpenAI's `POST /v1/images/generations`. Centralizes auth (reads `OPENAI_API_KEY` from `.env`), the curl call, base64→PNG decoding (jq path with a Python fallback for Git Bash on Windows), and post-write verification. Hard-codes `model: gpt-image-2` per user direction.
- **`yuval` agent** at `.claude/agents/yuval.md` — Hebrew creative-visual specialist with a hybrid layout: the canonical agent file lives flat under `.claude/agents/` (Claude Code only auto-discovers flat files there), while a separate `yuval/` working directory at the project root holds `reference/` (style inspiration), `outputs/` (generated PNGs + sibling `.txt` prompt files), and two human-pointer docs (`agent.md`, `skill.md`) that explicitly say "edit the canonical files, not me". Yuval's 6-step workflow: scan reference → extract style → pick relevant elements → write English prompt → call `gpt-image-gen` → save prompt sidecar + verify.
- **Reuven updated** — new section "סוכני המשנה תחת פיקודך / Sub-Agents Under Your Command" registers yuval with Hebrew + English trigger keywords. Yuval is explicitly **off-pipeline** (not part of the fixed agent-1..4 chain) — image requests bypass the content pipeline entirely. Iron rule #2 ("סדר הפייפליין קבוע") is preserved by design: it applies to the content pipeline only; yuval is a parallel track.

## Open Questions

- **Model name reality check.** User asked for `gpt-image-2` verbatim and confirmed it. As of training cutoff Jan 2026 this assistant only knew `gpt-image-1`; if the API returns 400 on `gpt-image-2`, the fix is a one-line update inside the skill's curl payload + the JSON-builder fallback.
- **Yuval E2E smoke test still untested.** All scaffolding verified by `ls`/`grep` but no live image was generated this session. Next session should: drop a tiny reference image (or empty `reference/`), invoke yuval with a simple Hebrew request, confirm a PNG lands in `outputs/` with a non-zero size and a sibling `.txt` exists.
- **Reuven routing not yet exercised.** Reuven's new sub-agent section says "dispatch yuval directly via Task" but no test run has confirmed Reuven actually picks it up when the user types a Hebrew trigger keyword. Test by asking Reuven a "תמונה של ..." request.
- **Tools list audit (carried over from [[reuven-agent-creation]]).** Still unresolved — Reuven's tools include Bash/Edit which conflict with "no execution". Re-evaluate after first real pipeline run.
- **Reference scan upper bound.** Yuval's prompt says "max 10 reference images, pick first 5–10". When the directory grows past that, do we want curated subsets or always read all? Defer until references actually pile up.

## Session Log

### 2026-05-06 — yuval + gpt-image-gen shipped [shipped]

- **What was done:**
  - Created `.claude/skills/gpt-image-gen/SKILL.md` — full procedure (load `.env` → build JSON safely with jq-or-python → POST to API → decode b64 → verify non-empty file). Documents both the project-spec minimal curl recipe and the standalone Python fallback.
  - Added `OPENAI_API_KEY` block to both `.env` and `.env.example`. User filled the real key into `.env` (gitignored, won't ship).
  - Scaffolded `yuval/` work dir at project root: `reference/.gitkeep`, `outputs/.gitkeep`, plus `yuval/agent.md` and `yuval/skill.md` as human-pointer docs that loudly say "the canonical file lives elsewhere — don't edit me".
  - Created `.claude/agents/yuval.md` — Hebrew system prompt with 6-step workflow, fixed reporting format, iron rules. Tools: `Read, Write, Glob, Grep, Bash`. Model: `sonnet`.
  - Updated `.claude/agents/reuven.md` — appended "Sub-Agents Under Your Command" section with yuval entry (Hebrew + English trigger keywords, Task dispatch instruction, off-pipeline note) plus a "Status" section dating today's wiring.
- **Decisions:**
  - **Hybrid yuval layout** (flat agent file + separate work directory) chosen because Claude Code does not auto-discover sub-folders under `.claude/agents/` — confirmed by reuven.md being a single flat file. The `yuval/` work dir at project root holds the runtime artifacts (reference images, outputs) that benefit from being co-located, with explicit pointer docs to prevent future drift.
  - **Model = `gpt-image-2` verbatim** per user (user confirmed via AskUserQuestion). Recorded as Open Question because this assistant's knowledge cutoff (Jan 2026) only knew `gpt-image-1`.
  - **Auto-run, no confirmation prompt** before API calls — user explicitly chose this over a "confirm before spending" flow. Cost per image is small ($0.01–0.04) and friction outweighs the saving.
  - **Yuval is off-pipeline.** Reuven's new section states clearly that yuval is dispatched **directly** when a visual trigger is detected, NOT routed through agent-1..4. This preserves Reuven's iron rule #2 ("fixed pipeline order") for the content pipeline while giving the visual track its own lane.
  - **Trigger keywords bilingual.** Hebrew set drawn from common phrasings ("תמונה של", "ציור של", "צייר", "באנר", "thumbnail"...); English set covers the same surface area ("generate image", "create image", "thumbnail", "banner"...). User specifically asked for both languages.
  - **Prompt-sidecar `.txt` per image** for iteration. Without storing the exact prompt that produced an image, you can't reliably tweak-and-regenerate — only re-roll from scratch.
- **Notes / Caveats:**
  - **`.env` now contains a real `OPENAI_API_KEY`** (user pasted it post-edit). `.gitignore` covers `.env` so it won't be committed; `.env.example` only carries the empty template.
  - **`jq` is the primary decode path; Python is the fallback.** Git Bash on Windows ships without jq by default, so the SKILL.md procedure tries jq first and falls back to a one-line Python invocation if jq is absent.
  - **Reuven dispatching yuval not tested live.** The wiring is in the agent file but a live `Task(subagent_type=yuval)` invocation hasn't been exercised yet — that's the next-session smoke test.
  - **Tension with reuven's "no execution" rule.** Yuval is registered as a sub-agent of Reuven's, but yuval *does* execute (it runs Bash, writes files, calls APIs). This is intended — yuval is a worker, Reuven is the dispatcher. The "no execution" rule applies to **Reuven**, not to his sub-agents.
- **Related:** [[reuven-agent-creation]], [[plugin-installations]], [[obsidian-vault-workflow-bootstrap]]
