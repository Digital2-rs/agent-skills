# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A distribution repo for shareable [Agent Skills](https://agentskills.io) targeting Laravel + Inertia/React/TS projects. Pure Markdown — no build system, no dependencies. Skills are consumed in other projects via Laravel Boost (`php artisan boost:add-skill Digital2-rs/agent-skills --skill=<name>`).

## Layout

Each skill is a folder under `skills/<skill-name>/`:
- `SKILL.md` — YAML frontmatter (`name`, `description`) + condensed instructions.
- `references/` or `rules/` — full detail, read on-demand.

`boost:add-skill` installs **skills only** (not the companion `.ai/guidelines/` pointer or `.claude/agents/` reviewer mentioned in README).

## SKILL.md conventions

- **Keep the SKILL.md body lean.** It is front-loaded into context, so it holds only condensed rules and a quick reference. Push full detail into `references/*.md` / `rules/*.md` that the agent opens on-demand. Don't bloat the body.
- **`description` is the trigger.** It must spell out exactly when the skill applies (keywords, file types, URL shapes, field names). That string is what makes the skill auto-invoke — be specific, not generic.
- **Field names / keys / commands are exact.** Match API keys, identifiers, and commands verbatim — no paraphrasing.

## Editing detail files

When a skill's behavior changes, update both the condensed `SKILL.md` body **and** the matching `references/`/`rules/` file so they stay consistent.
