---
name: laravel-conventions
description: Project conventions for writing Laravel (PHP) + Inertia/React/TS code — naming, Actions, DTOs, Query objects, models, migrations, code style. Use whenever creating or modifying PHP classes (Actions, Controllers, Models, Requests, Services, Queries, DTOs, Enums, Jobs, migrations) or Inertia/React components, or when building any new feature.
---

# Laravel conventions
Follow the project's conventions when writing or changing Laravel/Inertia code. The full, authoritative reference is `references/conventions.md` (in this skill folder) — read it when you need detail or an exact example. The condensed rules below cover the common cases.

## Before writing code
Use Laravel **Boost** tools: `search-docs` (version-specific; broad topic queries, no package names), `database-schema` before touching tables, `tinker`/`database-query` to inspect data. Don't guess APIs.

## Naming
- **Actions**: `Verb + Noun + Action` (`CreateAddressAction`), grouped by domain (`Actions/Addresses/`).
- **Controllers**: `Name + Controller`; non-CRUD action → separate **invokable** controller with `__invoke()`. Methods stay within CRUD verbs.
- **Middleware**: descriptive phrase, **no** `Middleware` suffix. **Services**: named after the system, **no** `Service` suffix. **Jobs**: describe the action, **no** `Job` suffix.
- **Enums**: `PascalCase` cases; name without suffix.

## Classes
- **Actions**: one public method `handle()`; constructor DI; returns model/DTO (no HTTP); wrap mutations in `DB::transaction()`. Helpers `protected`, but prefer extracting to a separate Action.
- **DTOs**: Spatie Data (`extends Data`), `Data` suffix, typed `public readonly` promoted props, built via `::from()`. No raw arrays into Actions.
- **Query objects**: `builder(array $filters = []): Builder` — return the Builder, not a collection.

## Models
- Casts via `casts()` method (not `$casts`). Local scopes via `#[Scope]` (no `scope` prefix). One trait per line.
- **No `$fillable`/`$guarded`** — models are unguarded via `nunomaduro/essentials`; safety is at the input boundary.

## Migrations
- **No `down()`** (forward-only). `foreignId(...)->index()` **without** `->constrained()`.

## General style
- Config via `#[Config('...')]`. Constructor property promotion (one per line, trailing comma). Validation rules as arrays, not pipes. Nullable `?Type`; explicit `: void`. `if`: braces always, happy path last, avoid `else`, separate `if`s over compound `&&`. `JsonResource::withoutWrapping()` is enabled.

## Frontend
- `.tsx` filenames kebab-case; page-scoped components colocated under the page; UI from shadcn/ui.

## After implementing
Hand the diff to the **convention-reviewer** subagent to check it against these rules before considering it done.