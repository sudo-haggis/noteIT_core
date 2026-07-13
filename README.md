# noteIt

A personal note-taking system built on plain Markdown, replacing Notion.

## What it does

Notes are organised using the [PARA method](https://fortelabs.com/blog/para/) — **P**rojects, **A**reas, **R**esources, **A**rchive — with each top-level folder holding sub-folders per project/area/topic, and individual notes as `.md` files within those.

Everything stays plain text and portable: no proprietary formats, no database, just Markdown files with YAML frontmatter for metadata (priority, status, dependencies, tags). Tasks use standard GFM checkboxes (`- [ ]`), plus lightweight `TODO-N:` inline markers for action items that don't need a full checkbox.

The vault is edited directly (e.g. via [Obsidian](https://obsidian.md/) on desktop and Android) and kept in sync across devices by the separate [noteIt_sync](https://codeberg.org/sudo-haggis/noteIt_sync) script — see that repo for the sync mechanism and per-device setup.

Claude Code skills (`new-note`, `process-inbox`, `seed-project`, `todo`, etc.) provide terminal-driven interaction with the vault — see `CLAUDE.md` for the full project context and skill reference.
