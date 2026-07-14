# Command: seed-project

Convert an idea brief file into a structured multi-phase PARA project.
Creates a project sub-folder with a parent index and numbered phase notes,
all dependency-wired in a sequential chain.

## Usage

```
/seed-project <path-to-brief>
```

Examples:
```
/seed-project _briefs/noteIt-skill-overhaul.md
/seed-project _briefs/2026-06-30_ccna-study-plan.md
```

---

## What to do

### Step 1 — Read and parse the brief

Read the file at the given path. Use your intelligence to extract the following — the
brief may be freeform, structured, or anywhere in between:

- **workspace**: look in YAML frontmatter first. If not there, look in the text for
  mentions of an existing workspace name.
- **priority**: look in YAML frontmatter. Integer `1`-`9`. Default to `6` if not found anywhere.
- **due**: look in YAML frontmatter. Optional — omit entirely from written notes if not found.
- **project title**: the H1 heading (`# Title`). If absent, derive from the filename
  by stripping a date prefix (e.g. `2026-06-30_`) and converting hyphens/underscores to
  title-cased words.
- **phases**: any ordered list in the document — numbered list, bulleted list, a
  `## Phases` section, or a clearly enumerated prose passage. Each item becomes one
  phase note. Preserve the order exactly.
- **goal text**: the content of a `## Goal` section, or the opening paragraph if no
  explicit section exists. May be absent — that's fine.
- **starting state text**: the content of a `## Starting State` section, or a
  `## Context` section (older brief format) if that's what's present instead. May be absent.
- **approach text**: the content of an `## Approach` section. May be absent.
- **problems text**: the content of a `## Problems` or `## Problems & Dependencies`
  section. May be absent.

**Abort immediately with a clear message if:**
- `workspace` cannot be determined → "Could not identify workspace. Add `workspace:` to
  frontmatter or mention an existing workspace name in the brief."
- The workspace value does not match an existing top-level vault folder → "Workspace
  '<value>' does not exist. Run /new-workspace first or check casing (Acme, Study, etc.)."
- No phases can be identified → "Could not find any phases. Add a list of phases to
  the brief, e.g. under a ## Phases heading."
- The given path ends in `_template.md` → "That is the template file. Copy it to a new
  file first, then run /seed-project on the copy."

---

### Step 2 — Derive slugs

- **project-slug**: take the project title, lowercase everything, replace spaces and
  special characters with hyphens, collapse multiple hyphens, strip leading/trailing
  hyphens. Example: "CCNA Study Plan 2026" → `ccna-study-plan-2026`
- **phase-slug-N**: apply the same rule to each phase name. Additionally strip any
  leading phase/step prefix the user may have typed, such as:
  `"Phase 1: "`, `"1. "`, `"Step 1 — "`, `"1) "` etc.
  Example: "Phase 1: Research and gather lab materials" → `research-and-gather-lab-materials`

---

### Step 3 — Collision check

- If `<workspace>/projects/<project-slug>/` **directory** already exists → abort:
  "Project folder '<workspace>/projects/<project-slug>/' already exists.
  Aborting to avoid overwrite."
- If `<workspace>/projects/<project-slug>.md` **flat file** exists → warn:
  "A flat note at '<workspace>/projects/<project-slug>.md' already exists.
  The new sub-folder project will not conflict with it, but you may want to reconcile
  them. Continue? (y/n)"
  Wait for the user to confirm before proceeding.

---

### Step 4 — Preview and confirm

Before writing any files, show the user exactly what will be created:

```
Ready to create:

  <workspace>/projects/<project-slug>/index.md         ← parent project index
  <workspace>/projects/<project-slug>/01-<slug-1>.md
  <workspace>/projects/<project-slug>/02-<slug-2>.md
  ...

Dependency chain:
  Phase 1 depends on: index
  Phase 2 depends on: index, Phase 1
  Phase 3 depends on: index, Phase 2
  ...

Create this structure? (y/n)
```

Do not write any files until the user confirms with `y`.

---

### Step 5 — Write the parent index.md

Path: `<workspace>/projects/<project-slug>/index.md`

```markdown
---
title: "<project title>"
created: <today YYYY-MM-DD>
updated: <today YYYY-MM-DD>
workspace: <workspace>
para: projects
status: not-started
priority: <priority>
tags:
  - para/projects
  - status/not-started
  - priority/<priority>
depends-on: []
blocked-by: []
sub-projects:
  - <workspace>/projects/<project-slug>/01-<phase-1-slug>
  - <workspace>/projects/<project-slug>/02-<phase-2-slug>
  - ...
---

# <project title>

## Starting State

<starting state text from brief, or the placeholder comment below if none was found>
<!-- What exists today? What does this project NOT do yet? -->

## Goal

<goal text from brief, or the placeholder comment below if no goal was found>
<!-- What does success look like when this project is complete? -->

## Approach

<approach text from brief, or the placeholder comment below if none was found>
<!-- How do we plan to get there? What tools, skills, or scripts are involved? -->

## Phases

- [[<workspace>/projects/<project-slug>/01-<phase-1-slug>|Phase 1: <phase 1 name>]]
- [[<workspace>/projects/<project-slug>/02-<phase-2-slug>|Phase 2: <phase 2 name>]]
- ...

## Problems & Dependencies

<problems text from brief, or the placeholder comment below if none was found>
<!-- What's going to stop us, and what needs to happen first? Include anything
     elsewhere in the vault this depends on or is affected by. -->

> [!info]- Dependencies
> <!-- Add [[wikilinks]] here when depends-on or blocked-by is populated -->
```

Rules:
- All four section headings (`Starting State`, `Goal`, `Approach`, `Problems & Dependencies`)
  are always written — this is a living structure meant to be filled in over time, not
  something to omit when empty.
- If text for a given section wasn't found in the brief, write its placeholder comment
  only — do not invent content.
- Phase wikilink labels must use the format `Phase N: <phase name>` (1-indexed).
- `sub-projects:` paths use vault-relative format without `.md` extension, same as `depends-on`.

---

### Step 6 — Write each phase note

For each phase numbered N (1-indexed), path:
`<workspace>/projects/<project-slug>/0N-<phase-slug>.md`

Zero-pad the number to 2 digits: `01`, `02`, ... `09`, `10`, `11`, etc.

**Frontmatter — Phase 1:**
```yaml
---
title: "<phase 1 name>"
created: <today YYYY-MM-DD>
updated: <today YYYY-MM-DD>
workspace: <workspace>
para: projects
status: not-started
priority: <same priority as brief, default 6>
tags:
  - para/projects
  - status/not-started
  - priority/<priority>
depends-on:
  - <workspace>/projects/<project-slug>/index
blocked-by: []
---
```

**Frontmatter — Phase N (N > 1):**
```yaml
depends-on:
  - <workspace>/projects/<project-slug>/index
  - <workspace>/projects/<project-slug>/0(N-1)-<prev-phase-slug>
```

**Body (all phases):**
```markdown
# <phase name>

<!-- What needs to happen in this phase? -->

## Notes

<!-- Body content here -->

> [!info]- Dependencies
> - [[<workspace>/projects/<project-slug>/index|<project title>]]
```

For phases N > 1, add the preceding phase wikilink on a second line in the callout:
```markdown
> [!info]- Dependencies
> - [[<workspace>/projects/<project-slug>/index|<project title>]]
> - [[<workspace>/projects/<project-slug>/0(N-1)-<prev-phase-slug>|Phase N-1: <prev phase name>]]
```

The callout must mirror `depends-on` exactly — one wikilink per entry, same paths.

---

### Step 7 — Confirm to user

After all files are written, output:

```
Created:
  ✓ <workspace>/projects/<project-slug>/index.md
  ✓ <workspace>/projects/<project-slug>/01-<slug-1>.md
  ✓ <workspace>/projects/<project-slug>/02-<slug-2>.md
  ...

Dependency chain wired:
  Phase 1 depends on: index
  Phase 2 depends on: index, Phase 1
  ...

Next steps:
  - Open index.md and fill in the Goal / Context sections if needed
  - Set status on each phase note as you begin work
  - Archive or delete _briefs/<brief-filename>.md if no longer needed
  - See .claude/skills/note-schema.md for field reference
```

---

## Rules

- Never create notes outside `<workspace>/projects/<project-slug>/`
- `index.md` is always the parent note — never named after the project slug
- Phase files are always zero-padded two digits: `01-`, `02-`, ... `09-`, `10-`, `11-`
- Every phase `depends-on` the parent index (all phases) plus the immediately preceding
  phase (phases 2+)
- `sub-projects:` in index frontmatter uses vault-relative paths without `.md` extension
  — same format as `depends-on`
- The `> [!info]- Dependencies` callout in each phase note must mirror its `depends-on`
  frontmatter exactly — one wikilink per path
- Wikilink labels for phases use the format `Phase N: <phase name>` consistently across
  index.md and phase note callouts
- Never process `_briefs/_template.md` — abort if that path is given
- Refer to `.claude/skills/note-schema.md` for the full field reference and allowed values
