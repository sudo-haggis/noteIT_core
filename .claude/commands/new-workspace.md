# Command: new-workspace

Create a new top-level workspace with a PARA folder structure.

## Usage

/new-workspace <name> [--pa]

- `<name>` is the workspace name. Use the exact casing the user provides.
- `--pa` (optional) creates a lite workspace with only `projects/` and `archive/`. Omit for the full four-folder PARA layout.

## Variants

There are two workspace variants:

| Variant | Folders | When to use |
|---------|---------|-------------|
| Full PARA (default) | projects, areas, resources, archive | Standard workspaces with ongoing responsibilities and reference material |
| PA (`--pa`) | projects, archive | Lean idea-capture workspaces — no ongoing areas or reference topics |

## What to do

### Full PARA (no flag)

1. Create the following folder structure at the vault root:

```
<name>/
  projects/
  areas/
  resources/
  archive/
```

2. Place a `.gitkeep` file inside each of the four PARA folders so git tracks them.

3. Create a `<name>/index.md` with this content (replace `<name>` with the actual workspace name, and `<today>` with today's date in `YYYY-MM-DD` format):

```markdown
---
title: "<name>"
created: <today>
updated: <today>
workspace: <name>
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

# <name>

## What is this workspace?

<!-- Describe the purpose of this workspace in 1-2 sentences -->

## PARA

| Folder | Purpose |
|--------|---------|
| `projects/` | Active work with a defined outcome and deadline |
| `areas/` | Ongoing responsibilities with no end date |
| `resources/` | Reference material organised by topic |
| `archive/` | Inactive items moved out of the above three |

> [!info]- Dependencies
> <!-- Add [[wikilinks]] here when depends-on or blocked-by is populated -->
```

### PA variant (`--pa`)

1. Create the following folder structure at the vault root:

```
<name>/
  projects/
  archive/
```

2. Place a `.gitkeep` file inside each of the two folders so git tracks them.

3. Create a `<name>/index.md` with this content:

```markdown
---
title: "<name>"
created: <today>
updated: <today>
workspace: <name>
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

# <name>

## What is this workspace?

<!-- Describe the purpose of this workspace in 1-2 sentences -->

## PARA (lite)

| Folder | Purpose |
|--------|---------|
| `projects/` | Active items — being tracked or considered |
| `archive/` | Inactive items or items promoted to a full workspace |

> [!info]- Dependencies
> <!-- Add [[wikilinks]] here when depends-on or blocked-by is populated -->
```

4. Confirm to the user what was created and where.

## Rules

- Never hardcode a specific workspace name inside this command file.
- Folder names inside the workspace are always lowercase: `projects`, `areas`, `resources`, `archive`.
- Do not create any notes or sub-folders beyond the structure above — keep it minimal.
