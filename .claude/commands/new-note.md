# Command: new-note

Scaffold a new note in the correct PARA location with standard frontmatter.

## Usage

```
/new-note <workspace> <para-category> <slug> [--priority 1-9] [--due YYYY-MM-DD] [--dated]
```

Examples:
```
/new-note Acme projects shopify-returns
/new-note Study resources bash-scripting --priority 8
/new-note Acme projects ccna-exam-prep --priority 2 --due 2026-08-01
/new-note Acme projects 2026-06-23_kickoff-call --dated
```

## What to do

1. **Validate inputs**
   - `<workspace>` must match an existing top-level folder in the vault root
   - `<para-category>` must be one of: `projects`, `areas`, `resources`, `archive`
   - `<slug>` must be kebab-case — reject if it contains underscores, spaces, or uppercase letters
   - If `--dated` is passed, prepend today's date: `YYYY-MM-DD_<slug>.md`. Otherwise use `<slug>.md`.
   - Warn the user (but do not abort) if `<para-category>` is `archive` — notes are usually moved there, not created there

2. **Determine target path**
   ```
   <workspace>/<para-category>/<filename>.md
   ```

3. **Check for collision** — if the file already exists, warn and abort. Never overwrite.

4. **Derive title** from slug per `note-schema.md`'s "Deriving a title from a slug" rule:
   replace hyphens with spaces, title-case each word, but keep known acronyms properly
   capitalized (e.g. `shopify-api-order-download` → `Shopify API Order Download`, not
   `Shopify Api Order Download`). Example: `shopify-returns` → `Shopify Returns`

5. **Write the file** with this template (fill in all placeholders):

```markdown
---
title: "<derived title>"
created: <today YYYY-MM-DD>
updated: <today YYYY-MM-DD>
workspace: <workspace>
para: <para-category>
status: not-started
priority: <priority, default 6>
tags:
  - para/<para-category>
  - status/not-started
  - priority/<priority>
depends-on: []
blocked-by: []
---

# <derived title>

<!-- Purpose: what is this note for? -->

## Notes

<!-- Body content here -->

> [!info]- Dependencies
> <!-- Add [[wikilinks]] here when depends-on or blocked-by is populated -->
```

If `--due` was passed, add a `due: <date>` line to frontmatter right after `priority`. Omit the line entirely if `--due` was not passed — never write `due:` with an empty value.

6. **Confirm** to the user: file path created, frontmatter fields set, remind them to:
   - Fill in the `<!-- Purpose -->` comment
   - Populate `depends-on` / `blocked-by` if needed, and mirror those as wikilinks in the Dependencies callout

## Rules

- Never create a note at the vault root
- `slug` must be kebab-case — if the user passes snake_case or spaces, reject and suggest the kebab-case equivalent
- `created` is always today's date — never ask the user for it
- `status` always starts as `not-started` — never pre-set to any other value
- `priority` is an integer `1`-`9`, defaults to `6` if `--priority` flag is omitted
- `due` is only written if `--due` is passed — omit the field entirely otherwise, never leave it blank
- Never overwrite an existing file — abort with a clear error message
- The Dependencies callout is always scaffolded even if empty — the user fills it in later
- Refer to `.claude/skills/note-schema.md` for the full field reference and allowed values
