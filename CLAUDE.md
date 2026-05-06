# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

"The Five Agents" is a content-creation agent team. A main "CEO" agent orchestrates a team of specialized sub-agents that together produce content end-to-end. The CEO and the specific sub-agents will be defined incrementally — this file will grow as the team takes shape.

Status: bootstrap. No code or agents yet — only the directory scaffolding.

## `.claude/` layout

Project-specific Claude Code configuration lives under `.claude/`:

- `.claude/agents/` — sub-agent definitions for the content team (CEO + specialists). Empty for now.
- `.claude/skills/` — project-specific skills used by the agents. Empty for now.
- `.claude/commands/` — custom slash commands for this project. Empty for now.

When adding to any of these, follow the standard Claude Code conventions for that folder (one Markdown file per agent / skill / command, with the appropriate frontmatter).
