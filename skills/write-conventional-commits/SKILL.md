---
name: write-conventional-commits
description: Write, review, or normalize Git commit messages using the Conventional Commits specification. Use when proposing a commit message, committing staged or working-tree changes, rewriting commit history, reviewing commit-message quality, configuring release automation, or deciding the correct type, optional scope, breaking-change marker, body, and footers.
---

# Write Conventional Commits

Produce a commit message that communicates one coherent change and supports readable history, changelogs, and semantic-release tooling.

## Follow repository policy first

1. Read applicable repository instructions and inspect recent commit subjects for local conventions.
2. Honor repository-specific types, scopes, subject-length limits, issue syntax, and generated-file rules when they differ from this guide.
3. Inspect the actual diff before describing it. When committing, inspect the staged diff separately and describe only what the commit will contain.
4. Do not stage, commit, amend, rebase, or push unless the user explicitly authorizes that action.

## Use the message structure

```text
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

- Use lowercase for `type` and normally for `scope`.
- Separate the header, body, and footer block with blank lines.
- Write the description as a concise imperative phrase: `add`, `fix`, `remove`, not `added`, `fixes`, or `removed`.
- Do not capitalize or end the description with a period unless local history requires it.
- Keep the header within the repository limit; if none exists, aim for 72 characters or fewer without sacrificing meaning.
- Use a body when the motivation, constraints, migration path, or non-obvious behavior matters. Explain why and what changed, not a line-by-line implementation.
- Write footers as Git trailers, such as `Refs: #123`, `Closes: #123`, or `Co-authored-by: Name <email>`.

## Choose the type by user-visible intent

- `feat`: add a new capability or observable behavior.
- `fix`: correct faulty behavior.
- `docs`: change documentation only.
- `refactor`: restructure code without changing behavior or fixing a bug.
- `perf`: improve performance without changing intended behavior.
- `test`: add or correct tests without changing production behavior.
- `build`: change the build system or external dependencies.
- `ci`: change continuous-integration configuration or scripts.
- `style`: change formatting, whitespace, or other non-functional presentation of code.
- `chore`: perform maintenance that fits no more precise type.
- `revert`: revert an earlier commit.

Prefer `feat` or `fix` whenever they accurately describe the change; release tooling commonly maps them to minor and patch releases. Do not use `chore` as a catch-all when a more informative type applies. If a commit mixes unrelated intents, recommend splitting it instead of hiding the mix behind a vague subject.

## Add a scope only when useful

Use a short noun that identifies the affected subsystem, package, domain, or feature:

```text
fix(checkout): preserve the selected delivery address
feat(catalog): add filtering by availability
```

Omit the scope when the change is repository-wide, no stable scope exists, or adding one would merely repeat the type or description. Reuse established repository scopes rather than inventing synonyms.

## Mark breaking changes explicitly

Add `!` before the colon when consumers must take action, and explain the impact and migration in a `BREAKING CHANGE:` footer:

```text
feat(api)!: require pagination parameters

Reject unbounded collection requests to protect API capacity.

BREAKING CHANGE: Clients must send both `page` and `per_page`.
```

Use uppercase `BREAKING CHANGE:` exactly. A breaking `fix`, `refactor`, or other type is still breaking; do not relabel it as `feat` merely to signal release impact.

## Handle common cases

- For dependency updates, use `build(deps)` when they affect build or runtime dependencies; follow local automation conventions for lockfile-only changes.
- For a revert, use `revert: <original subject>` and identify the reverted commit in the body, for example `This reverts commit <hash>.`.
- For generated files, describe the source-level intent rather than listing generated artifacts.
- For merge commits or automated version commits, preserve the tool or repository convention unless the user asks to rewrite it.
- When given only a ticket title or prose request, treat the message as provisional and say that the diff must be inspected before committing.

## Review before returning

Confirm that the message:

- matches the actual committed diff and contains one main intent;
- uses the most specific valid type and an established scope, if any;
- states the outcome rather than implementation trivia;
- calls out every breaking change and required migration;
- includes issue references or attribution only when supported by the task context;
- contains no invented ticket numbers, authors, tests, or claims.

Return the complete proposed message in a plain text code block when the user asks for a commit message. If multiple messages are genuinely plausible, recommend one and briefly explain the alternatives outside the code block.
