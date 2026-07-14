# noteIt_core

## *DEEP BREATH* Welcome to the future, I apologise already, but here we go!
Sorry its Claude all the way from here on in this project!

A personal note-taking system built on plain Markdown, replacing Notion — and,
as far as I know, one of the first **codebases with no code**: every piece of
functionality here is a natural-language instruction file interpreted by
Claude Code, not a compiled or interpreted program.

## What it does

Notes are organised using the [PARA method](https://fortelabs.com/blog/para/) — **P**rojects, **A**reas, **R**esources, **A**rchive — with each top-level folder holding sub-folders per project/area/topic, and individual notes as `.md` files within those.

Everything stays plain text and portable: no proprietary formats, no database, just Markdown files with YAML frontmatter for metadata (priority, status, dependencies, tags). Tasks use standard GFM checkboxes (`- [ ]`), plus lightweight `TODO-N:` inline markers for action items that don't need a full checkbox.

## A codebase with no code

Open this repo up and there's no `src/`, no package manifest, no build step, no
runtime to `npm install`. What's here instead:

- **`CLAUDE.md`** — project context and conventions, read by Claude Code at the
  start of every session
- **`.claude/commands/*.md`** — skill definitions (`/new-note`, `/todo`,
  `/seed-project`, …), written as instructions for Claude Code to follow, not
  as source for a compiler
- **`.claude/skills/*.md`** — reference specs (schema rules, PARA structure)
  that the skills point back to

There's nothing to execute outside of an LLM agent. The "program" is a set of
prompts; the "runtime" is Claude Code itself. This repo is the *behaviour* —
what happens when you ask for a new note, or run `/todo`, or seed a project
from a brief. It has no opinion about, and no access to, any notes unless it's
pointed at a vault.

## Where the data lives

This repo is intentionally just the logic layer. The actual notes — the PARA
folders (`projects/`, `areas/`, `resources/`, `archive/`) and everything in
them — live in a **separate vault repo**, hosted on Codeberg, edited directly
via [Obsidian](https://obsidian.md/) on desktop and Android.

```
noteIt_core (this repo)     — skills + conventions, no data, on GitHub
    │
    │  Claude Code, run from inside a vault checkout, reads CLAUDE.md
    │  + .claude/commands here and applies them to...
    ▼
the vault (data)            — PARA folders, plain Markdown + frontmatter, on Codeberg
    │
    │  kept in sync across devices by...
    ▼
noteIt_sync                 — pull --rebase / commit / push script, separate repo
```

A device-local `~/bin/noteIt_sync` script (from the sibling `noteIt_sync`
repo) handles cross-device sync; the `toDoIt` Go binary (from a separate
`todoIt` repo) handles TODO-marker scanning. Both are external tools this
repo's skills shell out to — neither lives here, and neither is required to
understand what this repo *is*.

## Skills

| Skill | Status | Description |
|-------|--------|-------------|
| `new-note` | built | Scaffold a new note in the correct PARA location |
| `new-workspace` | built | Create a new workspace with full PARA folder structure |
| `process-inbox` | built | Route `INBOX.md` items into the vault |
| `seed-project` | built | Convert an idea brief into a multi-phase PARA project |
| `todo` | built | Scan vault for `TODO-N:` markers, grouped by project |
| `sync` | built | Run `noteIt_sync` (pull --rebase, commit, push) and report the result |
| `tasks` | planned | List all open GFM checkbox tasks (`- [ ]`) across all notes |
| `subtasks` | planned | Show sub-tasks under a given parent task or note |
| `weekly-review` | planned | Summarise open tasks and recent activity across PARA |

See `CLAUDE.md` for the full project context, and the individual files under
`.claude/commands/` for each skill's exact behaviour — the table above is a
summary, not the spec.

## Getting started

1. Clone this repo, and separately clone (or create) a PARA-structured vault.
2. Run Claude Code from inside the vault directory, with this repo's
   `.claude/` and `CLAUDE.md` present (symlinked or copied in) so the skills
   and conventions are picked up.
3. Install `noteIt_sync` and `toDoIt` on `PATH` if you want `/sync` and
   `/todo` to work.

## License

MIT — see [LICENSE](LICENSE).
