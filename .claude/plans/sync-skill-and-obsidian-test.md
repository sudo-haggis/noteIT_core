# Plan: `/sync` skill + obsidian workspace test

## Context

`noteIt_sync` is a new per-device git sync repo (`codeberg.org:sudo-haggis/noteIt_sync`).
It has three branches: `main` (template), `ubuntu` (this machine), `android` (phone/Termux).
The script does: pull --rebase → add -A → commit (timestamped) → push.

This plan:
1. Installs the script to `~/bin/` so it's callable system-wide
2. Adds a `/sync` Claude skill so Claude can trigger a vault sync
3. Adds per-device cron config files to each branch of the noteIt_sync repo
4. Updates `obsidian/projects/phone_scripts.md` to reference the android branch
5. Creates a new `obsidian/projects/noteIt-sync-cron.md` project note
6. Adds TODO markers to both notes and runs `/todo --workspace obsidian` to test the stack

---

## Part 1 — Install to `~/bin`

Copy the ubuntu branch script to `~/bin/noteIt_sync` and chmod +x.

```bash
cp /path/to/noteIt_sync/noteIt_sync.sh ~/bin/noteIt_sync
chmod +x ~/bin/noteIt_sync
```

Verify path first — user said `../noteIt_sync` relative to noteIt vault. Check both:
- `~/workspace/codeberg/noteIt_sync/`
- `~/workspace/codeberg.org/sudo-haggis/noteIt_sync/`

`~/bin` is already on PATH (toDoIt lives there). No system changes needed — `~/bin` is the right call over `/bin` (avoids sudo, stays user-scoped).

---

## Part 2 — Cron config in noteIt_sync repo branches

Add two files to each device branch:

**`cron.conf`** — the raw cron entry for this device:
```
# noteIt sync — edit schedule as needed
*/30 * * * * ~/bin/noteIt_sync >> ~/.noteIt_sync.log 2>&1
```
(ubuntu branch uses `~/bin/noteIt_sync`; android branch uses Termux path)

**`install-cron.sh`** — one-shot installer:
```bash
#!/bin/bash
CRON_ENTRY=$(grep -v '^#' "$(dirname "$0")/cron.conf" | grep -v '^$')
( crontab -l 2>/dev/null; echo "$CRON_ENTRY" ) | crontab -
echo "Cron installed: $CRON_ENTRY"
```

This must be done on each branch separately (ubuntu branch gets ubuntu paths, android gets android paths). Commit to ubuntu branch first; android branch gets its own version.

Leave the schedule as `*/30` as a sensible default — the obsidian cron project note will have a TODO to decide final schedule per device.

---

## Part 3 — `/sync` Claude skill

New file: `.claude/commands/sync.md`

The skill:
1. Confirms vault location matches the branch's `NOTEIT_FILES_LOCATION`
2. Runs `noteIt_sync` (from `~/bin/noteIt_sync`)
3. Shows the git output — pull result, commit hash, push result
4. Reports success or surfaces the error clearly if something fails (conflict, auth issue, etc.)

No flags needed for now — it's a one-shot sync. Future: `--dry-run` flag could show what would be committed without pushing.

---

## Part 4 — Update `obsidian/projects/phone_scripts.md`

Current content: hardcoded bash sync script.

Replace with:
- Reference to the `noteIt_sync` repo on Codeberg
- Instructions to clone the `android` branch
- Note that the script handles pull/add/commit/push automatically
- Keep the Termux/widget context

Add:
```
TODO-1: clone android branch of noteIt_sync to phone and test
TODO-2: set up Termux widget to trigger noteIt_sync on demand
```

---

## Part 5 — New project: `obsidian/projects/noteIt-sync-cron.md`

Scaffold with `/new-note obsidian projects noteIt-sync-cron` conventions (manually, not via skill — no `--dated` flag needed).

Content covers:
- Goal: automate vault sync on all devices via cron/scheduler
- Per-device schedule decisions (ubuntu, android/Termux)
- How to use `install-cron.sh` from the device branch

Add:
```
TODO-1: decide sync frequency for ubuntu (currently set to */30)
TODO-2: research Termux cron equivalent for android (crond or Termux:Tasker?)
TODO: add log rotation for ~/.noteIt_sync.log
```

---

## Part 6 — Test the stack

Run `/todo --workspace obsidian` — should surface all the TODO markers from both notes,
grouped by priority, written to `obsidian/TODO.md`.

Verify:
- `obsidian/TODO.md` created with correct wikilinks
- P1 items appear before P2 and unranked
- Wikilinks resolve in Obsidian

---

## Files to create / modify

| File | Action |
|------|--------|
| `~/bin/noteIt_sync` | **Deploy** — copy from noteIt_sync repo ubuntu branch |
| `.claude/commands/sync.md` | **Create** — new skill |
| `noteIt_sync` repo, ubuntu branch | **Edit** — add `cron.conf` + `install-cron.sh` |
| `noteIt_sync` repo, android branch | **Edit** — add `cron.conf` + `install-cron.sh` (android paths) |
| `obsidian/projects/phone_scripts.md` | **Edit** — reference android branch, add TODOs |
| `obsidian/projects/noteIt-sync-cron.md` | **Create** — new project note |
| `obsidian/TODO.md` | **Generated** — by `/todo --workspace obsidian` |

---

## Verification

1. `noteIt_sync` runs from terminal without a path prefix — confirms `~/bin` install
2. `/sync` skill runs cleanly and shows git output
3. `obsidian/TODO.md` generated with correct P1/P2/unranked grouping and clickable wikilinks
4. All commits pushed to Codeberg
