# Plan: Scaffold New Workspaces & Projects

## Context
Expanding the noteIt vault with 4 new workspaces and 13 new project notes across new and existing workspaces. All notes follow the canonical frontmatter schema from `.claude/skills/note-schema.md`. Dependencies span across workspaces (Linux → Study → PPT). One commit per workspace to keep git log traceable.

## Typo resolutions
- `download_download_offline_maps` → `download_offline_maps`
- `arduio_finish_coding_challenges` → `arduino_finish_coding_challenges`
- `obsidian` — all lowercase as written

---

## Phase 1 — New workspace scaffolds (PARA dirs + .gitkeep + index.md)

| Workspace | Commit |
|-----------|--------|
| `LTDR/` | `feat(ltdr_workspace): scaffold LTDR PARA workspace` |
| `obsidian/` | `feat(obsidian_workspace): scaffold obsidian PARA workspace` |
| `Linux/` | `feat(linux_workspace): scaffold Linux PARA workspace` |
| `APOCOLYPS/` | `feat(apocolyps_workspace): scaffold APOCOLYPS PARA workspace` |

---

## Phase 2 — Project notes per workspace

### LTDR
- `LTDR/projects/website_text_into_notion.md`
- commit: `feat(ltdr_projects): add website_text_into_notion project`

### obsidian
- `obsidian/projects/widget_startup_scripts.md`
- `obsidian/projects/git_password_timeout_4hours.md`
- commit: `feat(obsidian_projects): add widget_startup_scripts and git_password_timeout_4hours projects`

### Linux
- `Linux/projects/align_file_systems.md`
- `Linux/projects/backup_fs_to_networked_drive.md`
- `Linux/projects/sys_baseline_add_pkg_managers.md`
- commit: `feat(linux_projects): add three Linux system projects`

### Study (existing)
- `Study/projects/windows_networked_drives.md` — depends-on: Linux/projects/backup_fs_to_networked_drive
- `Study/projects/ccna_challenge_breif.md`
- `Study/projects/arduino_finish_coding_challenges.md`
- `Study/projects/arduino_zig_led_lights.md`
- `Study/projects/arduino_zig_inputs.md`
- commit: `feat(study_projects): add five Study projects including cross-workspace dep`

### PPT (existing)
- `PPT/projects/pearson_network_drives.md` — depends-on: Study/projects/windows_networked_drives
- commit: `feat(ppt_projects): add pearson_network_drives with cross-workspace dep`

### APOCOLYPS
- `APOCOLYPS/projects/download_offline_maps.md`
- commit: `feat(apocolyps_projects): add download_offline_maps project`

---

## Frontmatter — project note template

```yaml
---
title: "Human Readable Title"
created: 2026-06-23
updated: 2026-06-23
workspace: <workspace>
para: projects
status: not-started
priority: medium
tags:
  - para/projects
  - status/not-started
  - priority/medium
depends-on: []
blocked-by: []
---

# Title

> [!info]- Dependencies
> <!-- [[wikilinks]] to dep notes — creates Obsidian graph edges -->
```

Notes WITH deps populate both:
- `depends-on:` frontmatter path (Dataview queries)
- Body callout wikilink (Obsidian graph view edges)

---

## Obsidian notes
- Dep paths in frontmatter = machine-queryable (Dataview)
- Dep wikilinks in body callout = graph edges in Obsidian
- Both must stay in sync — neither alone is sufficient

---

## Verification
- `find . -name "*.md" -not -path "*/.claude/*"` — confirm all files present
- `grep -r "depends-on" .` — confirm dep frontmatter correct
- Open Obsidian Graph View → confirm edges between dep notes
