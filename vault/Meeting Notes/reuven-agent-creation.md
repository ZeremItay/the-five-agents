# Reuven — CEO Orchestrator Agent Creation

## Overview

Implementation of PRD v1.0 (May 2026, author: Itay Zerem + Claude) for **ראובן**, the CEO orchestrator agent at `.claude/agents/reuven.md`. Reuven is the project's single front-door: the four sub-agents (`agent-1`..`agent-4`, defined in later PRDs) only run when Reuven dispatches them, in fixed pipeline order. Reuven himself never executes domain work — only orchestrates, reports, and synthesizes a unified final output. Status as of this entry: **wired but dormant** — the agent file exists, but Reuven won't be invoked automatically until `CLAUDE.md` is updated to route through him (explicitly out of scope per PRD §9).

## Open Questions

- Tools list audit. The agent currently lists `Task, Read, Write, Edit, Bash, Glob, Grep` per the PRD literal, but the "no execution" hard rule and the available tool set look in tension (`Bash`/`Edit` enable Reuven to do domain work directly). Re-evaluate after PRD §10 step 3 — mock-task testing — and trim if those tools go unused.
- When to update `CLAUDE.md` to actually route every task through Reuven? Today the agent file exists in isolation; the wiring change is the user's call (explicit PRD non-goal).
- When do PRDs for `agent-1`..`agent-4` land? Until they do, any pipeline run halts at the first dispatch with the dedicated "סוכן לא קיים" error.

## Session Log

### 2026-05-06 — Reuven created per PRD v1.0 [shipped]

- **What was done:** Wrote `.claude/agents/reuven.md` — full YAML frontmatter (`name: reuven`, `tools: Task, Read, Write, Edit, Bash, Glob, Grep`, `model: sonnet`) plus a Hebrew system prompt covering identity, five hard rules, the 5-step task-reception protocol, the agent-dispatch context template, the four user-reporting message types, the failure-handling decision tree, and the role boundaries. The prompt is in Hebrew throughout, in the "מנכ"ל ישראלי, בלי בולשיט" tone the PRD asks for.
- **Decisions:**
  - **Tools list verbatim from PRD** despite tension with the "no execution" rule. Marked as Open Question; will revisit after mock-task testing.
  - **Sub-agent names left as `agent-1`..`agent-4` placeholders.** PRD literal — replacing with real names is a future-PRD job.
  - **Defensive failure mode added** for "subagent doesn't exist yet": Reuven detects the Task error and immediately halts with a clear Hebrew message instead of retrying. This lets us mock-test Reuven before the four sub-agent files exist.
  - **`CLAUDE.md` left untouched** per PRD §9. Reuven exists but isn't auto-routed; the user will wire him in when ready.
- **Notes / Caveats:**
  - There's an architectural ambiguity in the PRD between "Reuven is a Task-dispatched subagent" (which can't pause for user input mid-flow) and "Reuven is the main agent's persona" (which can). The agent file works for both interpretations — final wiring is in `CLAUDE.md`, which is the user's territory.
  - Section 8.8 (PRD success criteria) requires the final output to be a unified synthesis, not 4 outputs glued together. This is encoded in step 5 of the task-reception protocol (`סנתז אותם לתוצר אחד עקבי`).
- **Related:** [[obsidian-vault-workflow-bootstrap]], [[plugin-installations]]

### 2026-05-06 — Yuval registered as off-pipeline sub-agent [shipped]

- **What was done:** Appended a new section to `reuven.md` — "סוכני המשנה תחת פיקודך / Sub-Agents Under Your Command" — registering `yuval` (creative/visual) with Hebrew + English trigger keywords. Also added a `## Status` section dating today's change. Reuven now knows to dispatch image requests **directly** to yuval via `Task(subagent_type=yuval)`, **bypassing** the agent-1..4 pipeline entirely.
- **Decisions:**
  - **Off-pipeline by design.** Yuval's section explicitly states he is NOT part of the fixed `agent-1`→`agent-2`→`agent-3`→`agent-4` chain. Iron rule #2 (fixed pipeline order) is preserved — it applies to the content pipeline only; visual requests are a parallel track.
  - **Bilingual triggers.** Hebrew keywords ("תמונה של", "ציור של", "באנר"...) and English keywords ("generate image", "thumbnail"...) both listed so Reuven can route from either language.
  - **In-file Status section** added per user request, even though `[[yuval-creative-agent-and-gpt-image-gen-skill]]` (the new vault note) carries the deeper history. Status is a high-level "what's wired today" snapshot.
- **Notes / Caveats:**
  - Tools-list audit (Open Question) still unresolved — Reuven's `Bash`/`Edit` are still in tension with "no execution". Will revisit after first real pipeline run.
  - Reuven's actual dispatch of yuval not tested live yet — see [[yuval-creative-agent-and-gpt-image-gen-skill#Open Questions]].
- **Related:** [[yuval-creative-agent-and-gpt-image-gen-skill]]
