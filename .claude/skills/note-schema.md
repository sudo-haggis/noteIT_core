# SKILL: Note Metadata Schema

## Purpose
This skill defines the canonical YAML frontmatter schema for all notes in this vault.
Every `.md` note (except `.gitkeep` and scratch files) should carry this header.

---

## Standard Frontmatter Template

```yaml
---
title: "Human readable title"
created: YYYY-MM-DD
updated: YYYY-MM-DD
workspace: <workspace-name>
para: projects
status: not-started
priority: 6
due: YYYY-MM-DD
tags:
  - para/projects
  - status/not-started
  - priority/6
depends-on: []
blocked-by: []
---
```

`due` is optional — omit the line entirely if there's no hard deadline, don't leave it blank.

---

## Field Reference

| Field | Required | Values |
|---|---|---|
| `title` | yes | Human-readable string. Separate from filename. |
| `created` | yes | ISO date. Set once at creation — never updated. |
| `updated` | yes | ISO date. Update whenever note content changes. |
| `workspace` | yes | Top-level workspace folder name: `Acme`, `Study`, etc. |
| `para` | yes | `projects` `areas` `resources` `archive` |
| `status` | yes (projects); optional (areas/resources) | See below. |
| `priority` | no (default `6`) | Integer `1`-`9`. `1` = most urgent, `9` = least. Same scale as `TODO-N:` markers, so a note's priority and its inline TODOs stay comparable. |
| `due` | no | ISO date. Omit entirely if there's no hard deadline. |
| `tags` | yes | Always include the three structural namespace tags. |
| `depends-on` | no | List of vault-relative paths. Empty list `[]` if none. |
| `blocked-by` | no | List of vault-relative paths. Empty list `[]` if none. |
| `sub-projects` | no | List of vault-relative paths to phase/child notes. Empty list `[]` or omit. Only used on project index notes created by `/seed-project`. |

### Status values

| Value | Meaning |
|---|---|
| `not-started` | Note exists, no work has begun |
| `next` | Queued up — the next thing to pick up, not yet actively worked |
| `in-progress` | Actively being worked on |
| `blocked` | Cannot proceed — populate `blocked-by` |
| `on-hold` | Paused deliberately |
| `complete` | Done — candidate for archive |
| `cancelled` | Dropped — candidate for archive |

---

## Tags Convention

Always include these three structural namespace tags in frontmatter:

```yaml
tags:
  - para/projects        # mirrors the `para` field
  - status/not-started   # mirrors the `status` field
  - priority/6           # mirrors the `priority` field
```

These are intentionally redundant with their scalar sibling fields:
- Scalar fields (`status: blocked`) are for Dataview table queries
- Namespaced tags (`status/blocked`) drive Obsidian's tag pane hierarchy

Add content-specific tags after the structural three:
```yaml
tags:
  - para/projects
  - status/in-progress
  - priority/2
  - tool/shopify
  - area/networking
```

---

## Dependency Paths

`depends-on` and `blocked-by` use **plain vault-relative paths** without `.md` extension
and without `[[wikilink]]` syntax:

```yaml
depends-on:
  - Acme/projects/shopify-checkout-dates
  - Acme/areas/operations

blocked-by:
  - Acme/projects/fulfilment-operation-script
```

**Why plain paths, not wikilinks?**
Obsidian does not resolve `[[wikilinks]]` in YAML frontmatter — they appear as raw strings
and do not create graph edges. Plain paths are simpler and grep-friendly.

### CLI grep examples

```bash
# All notes with dependencies in Acme
grep -r "depends-on:" Acme/

# All blocked notes across the vault
grep -r "status: blocked" .

# What is blocking Acme projects
grep -rn "blocked-by:" Acme/projects/

# All high-priority notes
grep -r "priority: high" .

# Notes depending on a specific note
grep -r "shopify-checkout-dates" .
```

---

## Obsidian Graph View — Body Dependency Section

Since frontmatter paths don't create graph edges, add a callout section to the note
body for any note that has `depends-on` or `blocked-by` entries:

```markdown
> [!info]- Dependencies
> - [[Acme/projects/shopify-checkout-dates|shopify checkout dates]]
> - [[Acme/areas/operations|Acme operations]]
```

- `> [!info]-` renders as a collapsed info callout in Obsidian reading view
- The `-` after the type keeps it collapsed by default (clean reading view)
- Wikilinks in the body DO create graph edges
- Keep this section in sync with the `depends-on` / `blocked-by` frontmatter fields

The `/new-note` command scaffolds this section automatically.

---

## Project Note Body Structure

Project notes (`para: projects` — both flat files and `/seed-project` sub-folder
`index.md`) use this four-section body structure. Treat it as a living document:
revisit and add to these sections whenever discussing "what should be done next"
on a project, rather than treating them as a one-time fill-in-the-blanks at creation.

```markdown
## Starting State

<!-- What exists today? What does this project NOT do yet? -->

## Goal

<!-- What does "complete" look like? A concrete finish line, not a direction. -->

## Approach

<!-- How do we plan to get there? What tools, skills, or scripts are involved?
     Use a GFM checkbox list here once the plan has concrete steps — this is
     the actual plan (see Inline TODO Markers below for how that differs from
     TODO-N: reminders). -->

## Problems & Dependencies

<!-- What's going to stop us, and what needs to happen first? Explain WHY —
     including anything elsewhere in the vault (other workspaces/projects) that
     this depends on or is affected by, not just this project in isolation.
     The depends-on/blocked-by fields + callout below are the machine-readable
     list; this section is the narrative behind them. -->

> [!info]- Dependencies
> <!-- Add [[wikilinks]] here when depends-on or blocked-by is populated -->
```

| Section | Purpose |
|---|---|
| `Starting State` | Current state / gap — what's missing or broken right now |
| `Goal` | Concrete definition of done |
| `Approach` | The plan — tools, skills, scripts; use checkboxes once steps are concrete |
| `Problems & Dependencies` | Narrative on blockers/prerequisites, including cross-project/workspace effects — pairs with `depends-on`/`blocked-by` frontmatter and the Dependencies callout |

This applies going forward — add it to existing project notes opportunistically as
they come up in conversation, no need to backfill the whole vault at once.

---

## Filename Convention

| Note type | Convention | Example |
|---|---|---|
| Evergreen (project/area index, reference) | `kebab-case.md` | `shopify-returns.md` |
| Timestamped (meeting note, daily log) | `YYYY-MM-DD_short-title.md` | `2026-06-23_kickoff-call.md` |

Use the `--dated` flag with `/new-note` to get a timestamped filename.

### Deriving a title from a slug

When a note's `title:` is derived automatically (from a slug or a routing tag, e.g. by
`/new-note` or `/process-inbox`), keep it short and sensible — a few title-cased words,
never a full sentence copy-pasted from source text.

Replace hyphens with spaces and title-case each word, **except** known acronyms and
technical terms, which keep their proper casing instead of being flattened:

| Slug word | Wrong | Right |
|---|---|---|
| `api` | Api | API |
| `gas` (Google Apps Script context) | Gas | GAS |
| `dx` | Dx | DX |
| `csv` | Csv | CSV |
| `pdf` | Pdf | PDF |

Example: `shopify-api-order-download` → `Shopify API Order Download`, not
`Shopify Api Order Download`. Extend this list as new acronyms come up — it isn't
exhaustive.

---

## Workspace Index Files

Workspace `index.md` files (e.g. `Acme/index.md`) use `para: areas` because a workspace
is an ongoing responsibility, not a bounded project.

```yaml
---
title: "Acme"
created: YYYY-MM-DD
updated: YYYY-MM-DD
workspace: Acme
para: areas
status: in-progress
priority: 6
tags:
  - para/areas
  - status/in-progress
  - priority/6
depends-on: []
blocked-by: []
---
```

---

---

## Inline TODO Markers

Use `TODO-N:` markers anywhere in a note body for lightweight action reminders.
These are found by the `/todo` skill via the `toDoIt` binary. Write them as a
GFM checkbox with the marker as the leading content:

```markdown
- [ ] TODO-1: confirm this approach before going further
- [ ] TODO-2: add error handling here
- [ ] TODO: tidy up wording later
```

Ticking the checkbox (`- [x]`) means done — `toDoIt` stops reporting that line
on future `/todo` scans. This is the canonical form going forward: the checkbox
gives the marker a real completion state instead of relying on someone
remembering to delete the line by hand.

Bare markers (no checkbox) still work for backward compatibility:

```markdown
TODO-1: confirm this approach before going further
```

...but have no completion state — resolving one means deleting the line, same
as before this convention existed. Prefer the checkbox form for anything new.

| Marker | Priority | `/todo` output tag |
|--------|----------|--------------------|
| `TODO-1:` | 1 — highest | `[P1]` |
| `TODO-2:` | 2 | `[P2]` |
| `TODO-N:` | N | `[PN]` |
| `TODO:` | unranked | `[--]` |

**These are not the same as GFM checkbox tasks used for real project steps:**
- `- [ ] task` — a step in the project's actual plan, formal, completion-tracked
- `- [ ] TODO-N: message` — a punctual reminder that happens to also be
  checkbox-shaped now, not a step in the plan itself. Keep the two conceptually
  separate even though they share syntax: bullets/checkboxes are *the plan*
  (decide → gather → do → finish, checked off in sequence), `TODO-N:` markers
  are things to come back to that hang off specific points in it — often
  time- or context-sensitive, not steps in the sequence themselves.

**`.todoignore` at vault root** excludes `.obsidian/`, `.claude/`, `.git/`, `_briefs/`
from all scans so markers in skill/config files are never surfaced.

---

## Project Sub-Folder Structure

When a project has multiple sequential phases, use a sub-folder instead of a flat file.
This structure is created by `/seed-project`.

### Folder layout

```
<workspace>/projects/<project-slug>/
  index.md               ← parent project note (always named index.md)
  01-<phase-1-slug>.md   ← Phase 1
  02-<phase-2-slug>.md   ← Phase 2
  03-<phase-3-slug>.md   ← Phase 3
```

### Naming rules

- The parent is always `index.md` — never named after the project slug
- Phase files are zero-padded two-digit numbers: `01-`, `02-`, ... `09-`, `10-`, `11-`
- Phase slugs follow the same kebab-case rule as all other note filenames

### `sub-projects:` field

The parent `index.md` carries a `sub-projects:` list in frontmatter:

```yaml
sub-projects:
  - Acme/projects/my-project/01-discovery
  - Acme/projects/my-project/02-build
  - Acme/projects/my-project/03-deploy
```

- Same path format as `depends-on`: vault-relative, no `.md` extension
- This field is optional and only meaningful on project index notes
- Existing flat notes without this field are unaffected

### Dependency chain

Each phase depends on the parent index and its immediately preceding phase.

| Note | `depends-on` |
|---|---|
| `index.md` | `[]` |
| `01-phase-one.md` | `[<project>/index]` |
| `02-phase-two.md` | `[<project>/index, <project>/01-phase-one]` |
| `03-phase-three.md` | `[<project>/index, <project>/02-phase-two]` |

Both `depends-on` frontmatter AND the `> [!info]- Dependencies` body callout must be
populated and kept in sync — the same dual-representation rule that applies to all
dependencies in this vault.

---

*This schema is the single source of truth for note metadata in this vault.*
*When in doubt, refer here before para-vault.md or CLAUDE.md.*
