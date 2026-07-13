# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# noteIt — Claude Code Context

## Project Overview

noteIt is a personal note-taking system designed to replace Notion.
Notes are stored as plain Markdown files, organised using the **PARA method**.
Claude Code skills are used to interact with notes from the terminal.

---

## PARA Structure

Notes follow the [PARA method](https://fortelabs.com/blog/para/) by Tiago Forte:

| Folder | Purpose |
|--------|---------|
| `projects/` | Active work with a defined outcome and deadline |
| `areas/` | Ongoing responsibilities with no end date (health, finances, home, etc.) |
| `resources/` | Reference material organised by topic |
| `archive/` | Inactive items moved out of the above three |

Each top-level PARA folder may contain sub-folders per project/area/topic.
Individual notes are `.md` files within those sub-folders.

### Note Naming Convention

| Note type | Convention | Example |
|---|---|---|
| Evergreen (project, area, resource index) | `kebab-case.md` | `shopify-returns.md` |
| Timestamped (meeting note, daily log) | `YYYY-MM-DD_short-title.md` | `2026-06-23_kickoff.md` |

Use the `--dated` flag with `/new-note` to get a timestamped filename.

---

## Claude Skills

| Skill | Status | Description |
|-------|--------|-------------|
| `new-note` | ✓ built | Scaffold a new note in the correct PARA location |
| `new-workspace` | ✓ built | Create a new workspace with full PARA folder structure |
| `process-inbox` | ✓ built | Route INBOX.md items into the vault |
| `seed-project` | ✓ built | Convert an idea brief into a multi-phase PARA project |
| `todo` | ✓ built | Scan vault for `TODO-N:` markers, grouped by project |
| `sync` | ✓ built | Run `noteIt_sync` (pull --rebase, commit, push) and report the result |
| `tasks` | planned | List all open GFM checkbox tasks (`- [ ]`) across all notes |
| `subtasks` | planned | Show sub-tasks under a given parent task or note |
| `weekly-review` | planned | Summarise open tasks and recent activity across PARA |

Skill implementations live in `.claude/commands/*.md`. Read the relevant command
file before changing a skill's behaviour — the descriptions above are a summary,
not the spec.

### External tooling

- **`toDoIt`** — standalone Go binary (developed in the separate `todoIt` repo,
  not part of this vault) that the `/todo` skill shells out to for scanning
  `TODO-N:` markers. It must be on `PATH`. `.todoignore` at vault root tells it
  which directories to skip (`.obsidian/`, `.claude/`, `.git/`, `_briefs/`, etc.)
  on vault-wide scans.
- **`noteIt_sync`** — a separate script (lives at `~/bin/noteIt_sync`, sourced
  from the sibling `noteIt_sync` repo) that the `/sync` skill runs. It validates
  the vault path/branch for the current device, then pulls, commits, and pushes.

---

## Metadata Schema

All notes carry standard YAML frontmatter (priority, status, dependencies, tags).
See `.claude/skills/note-schema.md` for the canonical field reference and allowed values.

Use `/new-note <workspace> <para-category> <slug>` to scaffold a note with correct frontmatter.

`.claude/skills/note-schema.md` and `.claude/skills/para-vault.md` are the
canonical, authoritative specs for schema and PARA rules respectively — if
anything here conflicts with them, they win.

---

## Capturing and Seeding Work

- **`INBOX.md`** (vault root) is the catch-all inbox. Anything above the
  `## Whiteboard` heading is a routable item (`/process-inbox` files each one
  into the vault using `@workspace #project-slug`/`!N`/`due:`/`dep:` tags —
  see `.claude/commands/process-inbox.md`). The `## Whiteboard` section itself
  is manual-only scratch space — never reorder, "clean up", or process it
  without being asked.
- **`_briefs/`** holds freeform idea briefs (see `_briefs/_template.md`).
  `/seed-project <brief>` turns one into a multi-phase project sub-folder
  (`index.md` + numbered, dependency-chained phase notes) — see
  `.claude/commands/seed-project.md`.

---

## Child Workspaces

A top-level workspace folder may carry its own `<workspace>/CLAUDE.md` for
subject matter that needs extra context beyond the standard PARA rules (e.g.
`linux_system/CLAUDE.md` for NAS/sysadmin work involving real remote devices).
A child `CLAUDE.md` **layers on top of this one** — it doesn't replace root or
global instructions, and typically adds workspace-specific guardrails (e.g.
"never SSH into a real device without separate explicit go-ahead").

---

## Vault Operational Notes

`.claude/memory/vault_observations.md` accumulates non-obvious operational
patterns discovered while working in this vault (e.g. cross-workspace
dependency chains, the dual frontmatter+wikilink representation of
dependencies, workspace-casing conventions, commit-granularity preferences).
Check it for context before making structural assumptions the notes
themselves don't state outright.

---

## Conventions

- All notes are Markdown (`.md`)
- Tasks use standard GFM checkbox syntax: `- [ ] task` / `- [x] done`
- Sub-tasks are nested with two-space indentation under a parent task
- Dates in notes use ISO 8601: `YYYY-MM-DD`
- Frontmatter `tags:` is primary for structural tags (`para/`, `status/`, `priority/` namespaces). Inline `#tag` in body is secondary for ad-hoc content tagging.
- No proprietary formats — everything stays plain text and portable

### Inline TODO Markers

Use `TODO-N:` markers in note bodies for action items that don't warrant a formal checkbox task.
Scanned by the `/todo` skill via the `toDoIt` binary.

| Marker | Priority | When to use |
|--------|----------|-------------|
| `TODO-1: message` | Highest | Must do before this note is useful |
| `TODO-2: message` | High | Should do soon |
| `TODO-N: message` | Nth | Lower urgency, open-ended scale |
| `TODO: message` | Unranked | Casual reminder, no urgency |

These are distinct from GFM checkbox tasks (`- [ ]`):
- Checkboxes = formal completion-tracked tasks (future `/tasks` skill)
- `TODO-N:` markers = lightweight action reminders, grep-found, no completion state
