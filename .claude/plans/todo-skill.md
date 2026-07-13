# Plan: `/todo` Claude Skill — toDoIt integration into noteIt

## Context

`toDoIt` is installed at `~/bin/toDoIt`. It greps files for `TODO:` and renders
coloured terminal output. The `/todo` skill will invoke this binary as its scan engine and layer
priority-sorted rendering on top.

The `todo` skill is already listed in `CLAUDE.md` as planned but not built — this delivers it.

Two task systems coexist with different purposes:
- `- [ ]` GFM checkbox = formal completion-tracked task (future `tasks` skill)
- `TODO-N:` inline marker = action/reminder (grep-found via toDoIt, no completion state)

---

## Marker Convention

| Marker      | Meaning                          |
|-------------|----------------------------------|
| `TODO-1: x` | Highest priority — shown first   |
| `TODO-2: x` | Second priority                  |
| `TODO-N: x` | Nth priority (open-ended scale)  |
| `TODO: x`   | Unranked — shown last            |

Numbers optional. Plain `TODO:` works for casual markers.

---

## How the skill uses toDoIt

The skill instructs Claude to:

1. Run `toDoIt -d` from the noteIt root directory (or a workspace subdir for `--workspace` filter)
   - `-d` removes the default 5-level depth cap
   - toDoIt respects `.todoignore` — we add one to exclude non-note dirs
2. Parse the output: extract file path, line number, and TODO text for each hit
3. From the TODO text, detect `TODO-N:` and extract priority number N
4. Re-render grouped by priority tier (1 first, unranked last), with workspace and file subheadings

---

## Files to Create/Modify

### NEW: `.todoignore` (noteIt root)
```
.obsidian/
.claude/
.git/
```

### NEW: `.claude/commands/todo.md`
Interface:
```
/todo [--workspace <name>] [--priority <N>] [--top]
```
- No args — scan entire vault, render all items grouped by priority
- `--workspace Acme` — run toDoIt from `Acme/` subdirectory only
- `--priority 1` — show only `TODO-1:` items
- `--top` — show only the single highest-priority tier that has any items

Output format:
```
## Priority 1
### Acme
- [Acme/projects/shopify-returns.md:14] Confirm API endpoint with team

## Unranked
### Linux
- [Linux/projects/sys_baseline.md:8] Add package manager notes
```

### MODIFY: `CLAUDE.md`
- Add **Inline TODO Markers** to Conventions section
- Update Planned Skills table: mark `todo` as implemented

### MODIFY: `.claude/skills/note-schema.md`
- Add inline TODO marker section (syntax, priority numbering, lives in body not frontmatter)

---

## What is NOT changing

- `toDoIt` binary and source — no modifications
- GFM checkbox tasks and future `tasks` skill — unchanged
- YAML frontmatter schema — no new fields needed

---

## Verification

1. Add `TODO-1: test high priority` to one existing note
2. Add `TODO: unranked test` to a different note
3. `/todo` — both appear, Priority 1 first
4. `/todo --priority 1` — only the priority-1 item
5. `/todo --workspace Linux` — only Linux items
6. `/todo --top` — only highest populated tier
