# SKILL: PARA Vault Management

## Purpose
This skill governs how Claude interacts with this Obsidian vault.
PARA is the structural law. Claude enforces it — not passively.

---

## PARA Categories

### 1. Projects (`/Projects`)
Active work with a **defined outcome and deadline**.
- Has a clear finish line
- Being worked on **now**
- Examples: `CAZ-app-launch`, `wtk-shopify-migration`, `ccna-exam-prep`

**Rules:**
- Every project folder must have an `index.md` with: goal, deadline, status
- No orphaned notes — everything links back to the project index
- When complete → move to `/Archive`, do not delete

### 2. Areas (`/Areas`)
Ongoing responsibilities with **no end date**.
- Standards to maintain, not goals to achieve
- Examples: `van-systems`, `health`, `finances`, `wtk-operations`

**Rules:**
- No tasks live here — tasks belong in a Project
- If an Area spawns active work → create a Project, link back to Area
- Reviewed periodically, not daily

### 3. Resources (`/Resources`)
Reference material. **No action required.**
- Topics of interest, research, how-tos, clippings
- Examples: `networking`, `bash-snippets`, `obsidian-tips`, `zig-notes`

**Rules:**
- Flat-ish structure — avoid deep nesting
- Never put active tasks or project notes here
- If a resource becomes actionable → spawn a Project

### 4. Archive (`/Archive`)
Completed or inactive items. **Cold storage.**
- Completed projects, dropped ideas, outdated references
- Mirror the original category folder structure inside Archive

**Rules:**
- Never delete — archive instead
- Archived items are read-only in practice
- Structure: `/Archive/Projects/`, `/Archive/Areas/`, `/Archive/Resources/`

---

## Note Metadata Schema

All notes must carry standard YAML frontmatter. See `.claude/skills/note-schema.md` for
the canonical field reference, allowed values, dependency path format, and grep examples.

Key points:
- `status`, `priority`, `para`, `workspace` are required scalar fields
- `tags` must always include `para/`, `status/`, `priority/` namespace tags
- `depends-on` and `blocked-by` use plain vault-relative paths (not wikilinks)
- Body must include a `> [!info]- Dependencies` callout when dependencies exist

---

## Obsidian Conventions

- **Folder names:** `kebab-case`, lowercase, no spaces
- **File names:**
  - Evergreen notes (project index, area, resource): `kebab-case.md`
  - Timestamped notes (meetings, daily logs): `YYYY-MM-DD_short-title.md`
- **Daily notes:** `/Daily/YYYY-MM-DD.md` — outside PARA, not in Projects
- **Templates:** `/Templates/` — outside PARA
- **Index files:** Every Project folder has `index.md` as the entry point
- **Wikilinks over markdown links** for internal notes
- **Tags:** Frontmatter `tags:` is primary — always include the three structural namespace tags (`para/`, `status/`, `priority/`). Inline `#tag` in body text is secondary, for ad-hoc content tagging only.

---

## Claude Behaviour in This Vault

### On every session start:
- Identify the working context — which PARA category are we in?
- Flag anything that looks misplaced before doing other work

### When creating notes:
- Always confirm the correct PARA location before writing
- Never dump a new `.md` file in vault root
- Create the `index.md` if starting a new Project

### When asked to "save" or "note" something:
- Determine: is this a task (Project), standard (Area), reference (Resource), or dead (Archive)?
- Place accordingly — ask if ambiguous

### Flags to raise:
- Notes sitting in vault root
- Project folders missing `index.md`
- Tasks or to-dos found inside `/Areas` or `/Resources`
- Active work found inside `/Archive`
- Folders with no clear PARA parent

---

## Common Traps — Do Not Fall Into These

| Trap | Reality |
|---|---|
| "It's a project" | Does it have a deadline and outcome? If not → Area or Resource |
| "I'll sort it later" | No. Place it correctly now or Archive it |
| "Just put it in Resources" | Resources are reference, not a junk drawer |
| Deep nested folders | Flat is better — if you need 3 levels, question the structure |
| Duplicate notes | Search before creating — link, don't duplicate |

---

## Quick Reference

```
vault/
├── Projects/        # Active, goal-driven, deadline-bound
├── Areas/           # Ongoing responsibilities
├── Resources/       # Reference, no action needed
├── Archive/
│   ├── Projects/
│   ├── Areas/
│   └── Resources/
├── Daily/           # Daily notes (outside PARA)
└── Templates/       # Outside PARA
```

---

*This skill is authoritative. When in doubt, enforce PARA over convenience.*
