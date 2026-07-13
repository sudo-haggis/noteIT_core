# Command: sync

Run the noteIt vault sync — pulls latest changes, commits any local edits, and pushes to remote.

## Usage

```
/sync
```

---

## What to do

### Step 1 — Run the sync script

Run:
```bash
noteIt_sync
```

The script lives at `~/bin/noteIt_sync`. It will:
- Validate the vault path and git branch are correct for this device
- `git pull --rebase` — fetch and rebase local commits on top of remote
- `git add -A` — stage all changes
- `git commit` — timestamped commit (`chore(phone_sync): YYYY-MM-DD HH-MM-SS`)
- `git push` — push to remote

### Step 2 — Report output

Print the full output from the script so the user can see exactly what happened.

Then summarise in one line:
- If push succeeded: `Sync complete — vault is up to date.`
- If nothing to commit: `Sync complete — nothing to commit, vault already up to date.`
- If the script exited early (wrong branch/path): `Sync skipped — <reason from script output>.`
- If any git command failed: `Sync failed — <error>. Check the output above.`

---

## Rules

- Never modify vault notes before syncing — run the script as-is
- Do not pass any extra flags to the script
- If the script reports a conflict or rebase failure, surface it clearly and tell the user
  to resolve it manually before running `/sync` again
