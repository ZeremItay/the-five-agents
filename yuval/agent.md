# yuval — Working Directory

This is **yuval's work directory**. It's a pointer-doc for humans, not a Claude Code agent definition.

The canonical agent definition lives at:

> [`.claude/agents/yuval.md`](../.claude/agents/yuval.md)

That's the file Claude Code loads when invoking the `yuval` sub-agent. **Do not** edit this `agent.md` expecting it to change yuval's behavior — it won't. Edit the file under `.claude/agents/` instead.

## What goes where

| Directory | Purpose |
|---|---|
| `reference/` | **Inspiration images** for visual consistency. Drop PNG / JPG / WEBP files here. Yuval scans this directory at the start of every request and uses it to derive style, palette, and composition cues. Single directory — do not rename or split into sub-folders. |
| `outputs/` | **Generated images** land here, named `<YYYY-MM-DD>-<slug>.png`. Each image has a sibling `.txt` with the exact prompt that was used (kept for iteration). |

## Workflow

1. Drop reference images into `reference/` (optional — if empty, yuval generates without style anchoring).
2. Ask yuval (via Reuven, or directly) for an image.
3. Yuval scans `reference/`, synthesizes an English prompt, calls the `gpt-image-gen` skill, and saves to `outputs/`.
4. Open `outputs/<filename>.png` to view; open the sibling `.txt` to see the prompt yuval used.

## Iterating on a result

If you want to tweak an existing image: open its `.txt`, copy the prompt, hand it back to yuval with the change you want. Yuval will treat it as a fresh request but with your starting prompt as scaffolding.
