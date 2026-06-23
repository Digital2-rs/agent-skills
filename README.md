# laravel-skills

Shareable AI agent skills for Laravel + Inertia/React/TS projects.
Follows the open [Agent Skills](https://agentskills.io) format.

## Install via Laravel Boost

```bash
php artisan boost:add-skill Digital2-rs/agent-skills --skill=laravel-conventions
# if skills weren't enabled at install time, sync them:
php artisan boost:install --skills --no-interaction
```

This downloads the skill to `.ai/skills/laravel-conventions/` and Boost syncs it
into your agent directory (e.g. `.claude/skills/laravel-conventions/`), bringing
`references/conventions.md` along with it.

## Repo layout

```text
laravel-skills/
├── README.md
└── skills/
    └── laravel-conventions/
        ├── SKILL.md                 # metadata (name, description) + short instructions
        └── references/
            └── conventions.md       # full conventions document (read on-demand)
```

## What's inside

- **laravel-conventions** — project conventions for writing Laravel (PHP) + Inertia/React/TS:
  naming, Actions, DTOs, Query objects, models, migrations, and general code style.
  The full reference (`references/conventions.md`) is read on-demand; the `SKILL.md`
  body holds a condensed version so the agent stays in convention without bloating context.

## Companion pieces (not installed by add-skill)

`boost:add-skill` installs **skills only**. For the full three-layer setup, copy these
into each consuming project:

- `.ai/guidelines/conventions.blade.php` — a tiny always-on pointer (Boost renders it into
  `CLAUDE.md` / `AGENTS.md`) that reminds the agent to use the `laravel-conventions` skill.
- `.claude/agents/convention-reviewer.md` — a read-only backstop subagent that reviews the
  diff against the conventions after a feature is implemented.

## Updating

The skill is regenerated in agent directories by Boost. To refresh after changes:

```bash
php artisan boost:update
```

## License

MIT