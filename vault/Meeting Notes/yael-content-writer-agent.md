# Yael (Content Writer Agent)

## Overview

Second specialist sub-agent under Reuven, alongside [[yuval-creative-agent-and-gpt-image-gen-skill|yuval]]. **Yael is LLM-only** — tools = `Read, Write, Edit, Glob, Grep` (no Bash, WebSearch, or API). She rewrites raw articles from `Content/` in the project's voice, drops `{{IMAGE_NEEDED: "..."}}` placeholders where images are needed, and saves drafts to `Output/`. **She does not call yuval directly** — Claude Code forbids sub-agents dispatching sub-agents, so the orchestrator pattern is mandatory: yael leaves placeholders, Reuven iterates them, dispatches yuval per placeholder, and substitutes Markdown image references. Reuven's prompt was extended with a 6-step image-substitution protocol that runs after every yael invocation.

New directory layout:

- `.claude/agents/yael.md` — canonical agent file (flat, sibling to `reuven.md` and `yuval.md`).
- `yael/` — working directory: `style-guide.md` (master style doc, currently a stub for the user to fill), `reference/` (example texts in our voice), `agent.md` (human pointer doc).
- `Content/` — raw articles waiting to be rewritten.
- `Content/Ready/` — copies of source articles that yael has already processed (yael writes a copy; Reuven removes the original from `Content/` after image substitution).
- `Output/` — final drafts (yael writes a draft with placeholders → Reuven overwrites with the substituted-images final).

## Open Questions

- **`yael/style-guide.md` is a stub.** Yael's iron rule #1 says "stop and report if style-guide is missing or empty" — but the file currently has only TODO/למלא markers. **First real run will halt at style-guide load** until the user populates it. This is intentional design — better to fail loudly than to invent a style.
- **`yael/reference/` is empty.** Same caveat — no example texts yet. Yael handles this gracefully (works from style-guide alone) but quality degrades without examples.
- **Source-removal seam.** Yael writes the source copy to `Content/Ready/<name>.md` but cannot `rm` the original from `Content/` (no Bash). Reuven does the deletion in step 4 of the image-substitution protocol. **Risk:** if Reuven crashes mid-flow, `Content/<name>.md` remains while `Content/Ready/<name>.md` already exists — manual cleanup needed. Document this if it bites.
- **Output filename collision policy.** If `Output/<name>.md` already exists from a previous run, yael will overwrite without warning. Likely fine for v1 (you'd usually want the latest version anyway), but consider a `--force` style guard if it bites.
- **Same Task-discovery constraint as yuval.** New agent files aren't auto-registered as `subagent_type` mid-session. Reuven won't be able to dispatch yael until a session restart.
- **No live test yet.** All scaffolding written but no end-to-end run (style-guide is a stub anyway). Next session: fill style-guide, drop a sample article in `Content/`, ask Reuven to "שכתב את Content/<name>.md".

## Session Log

### 2026-05-06 — yael shipped (LLM-only sub-agent + Reuven image-substitution protocol) [shipped]

- **What was done:**
  - Created `.claude/agents/yael.md` — full Hebrew agent prompt with 6-step workflow (input → load style → read article → rewrite with `{{IMAGE_NEEDED}}` placeholders → save to `Output/` → copy source to `Content/Ready/`), iron rules, fixed reporting format, and explicit "no Bash, no API, no sub-agent dispatch" boundaries. Tools: `Read, Write, Edit, Glob, Grep`. Model: `sonnet`.
  - Scaffolded `yael/` working dir: `style-guide.md` (stub with TODO sections — user will fill), `reference/.gitkeep`, `agent.md` (human pointer doc explaining the orchestrator pattern and why yael can't call yuval directly).
  - Scaffolded `Content/`, `Content/Ready/`, `Output/` at project root with `.gitkeep` stubs.
  - Updated `.claude/agents/reuven.md`: added a `### yael — Content Writer` block under "Sub-Agents Under Your Command" with bilingual trigger keywords (rewrite/edit/translate/summarize and Hebrew equivalents). Added a `#### פרוטוקול image-substitution לאחר yael` sub-section with the full 6-step Reuven flow: collect placeholders → loop (dispatch yuval per placeholder, build Markdown image with alt+caption, Edit-replace) → verify no leftovers via Grep → delete `Content/<name>.md` → user report → Obsidian log entry. Added explicit failure handling for partial yuval failures (keep placeholder, mark in report, don't delete source).
  - Appended a Status entry dating today's wiring.
- **Decisions:**
  - **Orchestrator pattern, mandatorily.** Yael cannot dispatch yuval — Claude Code subagents can't call subagents. Reuven owns the multi-stage flow. This was confirmed in the user's brief and is the central architectural fact of the design.
  - **Final-output overwrites the draft.** When Reuven finishes substituting images, he overwrites `Output/<name>.md` (yael's draft is gone). History lives in Reuven's vault log entry. Considered keeping a `Drafts/` sibling but the user picked the simpler overwrite — fewer paths, less branching.
  - **Reuven specifies the source filename in dispatch.** Yael does NOT scan `Content/`. The orchestrator hands her a path. Cleaner, batchable, and gives Reuven a chance to confirm with the user before any rewrite. Rejected alternatives: yael picks the alphabetically-first file, or yael processes the whole batch.
  - **Image placeholder format = `{{IMAGE_NEEDED: "<English prompt>"}}` on its own line.** Reuven substitutes with `![alt](path)` + italic caption (full Markdown image with caption). Rejected: bare `<img>` and `<figure>` HTML — Markdown is more portable across CMSs.
  - **Yael copies source instead of moving it.** Yael lacks Bash, so she can't `mv`. Workaround: yael writes a duplicate to `Content/Ready/`, Reuven `rm`s the original. Documented in both the yael agent prompt (as a "you can't do this — Reuven will") and in Reuven's image-substitution protocol step 4. Alternative considered: give yael Bash for filesystem ops only — rejected because it leaks past the "LLM-only" boundary.
  - **`yael/style-guide.md` shipped as a stub** rather than left missing. The user's brief said "אצור בנפרד" (I'll create separately). Stub provides scaffolding (sections for Tone / Length / Structure / Language / Phrases-we-like / Phrases-to-avoid) so when the user fills it they have a frame, not a blank page.
- **Notes / Caveats:**
  - **Two un-pushed commits ahead of `origin/main`** at this point: `4dd7e5e` (yuval+gpt-image-gen) and the new yael commit when it lands. Direct push to main is blocked by a guard rule. Both commits will need a feature-branch + PR (or a relaxed push permission) before they leave the local machine.
  - **`yuval/outputs/2026-05-06-bull.png`, `2026-05-06-bull.txt`, `2026-05-06-itay-on-bull.png`, `2026-05-06-itay-on-bull.txt`** — generated during yuval testing earlier today, not yet committed. Not part of this commit either; those are runtime artifacts and the user can decide whether to track them.
  - First Edit attempt on `reuven.md` failed because the string used in `old_string` was a slightly older version of yuval's description than what's currently in the file. Re-read the file and used a different unique anchor (the line ending in "...לדיווח שלך למשתמש." plus the `## Status` header) to land the yael block in the right place. Lesson: when a previous edit in the same session has changed text, re-read before the next Edit.
- **Related:** [[reuven-agent-creation]], [[yuval-creative-agent-and-gpt-image-gen-skill]]
