# Command: process-inbox

Read INBOX.md and file each item into the vault interactively.

## Usage

/process-inbox

No arguments. Processes all items above the `## Whiteboard` section one by one.

Example INBOX.md lines:
```
- Datepicker on checkout tied to line_items @Acme #shopify-checkout-datepicker !2
- Track DX consignment status @Acme_Fulfilment #gas-fulfilment-scrapper !5 due:2026-08-01
- Shopify API up and running, pulling into sheets @Acme_Fulfilment #gas-fulfilment-shopify !3 dep:Acme_Fulfilment/projects/shopify-api dep:Acme/projects/shopify-date-per-product
```

---

## Routing syntax (recognised in item text)

| Pattern | Meaning |
|---------|---------|
| `@workspace #project-slug` | File into an existing or new note at `<workspace>/projects/<project-slug>.md` |
| `@workspace` (no `#project`) | Create a new project note in `<workspace>/projects/`, deriving the slug from the item text |
| `!N` (e.g. `!3`) | Set `priority: N` (integer `1`-`9`). Omit for the schema default (`6`). |
| `due:YYYY-MM-DD` | Set the `due` field. Omit entirely if not present in the item — never write a blank `due:`. |
| `dep:<vault-relative-path>` | Add `<path>` (no `.md`, no leading slash — same format as `depends-on` elsewhere) to this note's `depends-on` list. Repeatable — use one `dep:` per dependency. |
| No `@workspace` tag | Ask the user where this item should go |

Matching is case-insensitive for `@workspace` and `#project-slug`. `@workspace` must be an
existing top-level vault folder. `#project-slug` is normalized to kebab-case before use
(lowercased, spaces/underscores collapsed to hyphens) regardless of how it was typed —
keeps filenames consistent with `note-schema.md`'s convention even from a quick mobile
capture where kebab-case is easy to forget.

`dep:` targets do not need to already exist yet — write the path as given. If it's another
item being filed in the same `/process-inbox` run, it's fine whether that item was
processed before or after this one; if it doesn't resolve to a real note, that's no
different from any other mistyped `depends-on` path in the vault and can be fixed by hand
later.

The `title:` is **never** the raw item text — it's derived from the `#project-slug` (or
the slug generated from the item text in Case B), same as `/new-note`: hyphens become
spaces, each word is title-cased, with known acronyms kept properly capitalized per
`note-schema.md`'s "Deriving a title from a slug" section (e.g. `shopify-api-order-download`
→ `Shopify API Order Download`, not a full sentence and not `Shopify Api Order Download`).
The original item text (minus tags) goes in the note body's `Starting State` section
instead, so detail isn't lost even though the title stays short.

---

## Processing logic

Work through each item in the main list (everything above `## Whiteboard`) one at a time.

### Before classifying: validate `@workspace`, don't take the tag at face value

Quick/mobile captures are typo-prone — `@workspace` tags written away from the vault
regularly don't match a real folder (`@van` instead of `van_chores`, `@CCNA` when there's
no such workspace but a `Study/projects/ccna.md` already exists, etc.). Before treating an
item as Case A or B:

1. Diff the given `@workspace` token against the actual top-level vault folders.
2. If it doesn't match exactly, don't just fall through to Case C's generic "where should
   this go?" — proactively suggest the nearest real workspace, and check whether an
   existing note already covers the topic (see the Case B collision check below). Present
   that as the first option, not an open-ended question.

### Case A — `@workspace #project-slug` present

1. Normalize `#project-slug` to kebab-case, then resolve `<workspace>/projects/<project-slug>.md`.
2. **File exists** → Tell the user the item and the existing note path. Ask:
   - **Open it** — open the note for the user to edit manually, then remove the item from INBOX.
   - **Leave in INBOX** — skip this item and leave it in the list untouched.
3. **File does not exist** → Create a new project note using the standard frontmatter template
   and the four-section project body structure (see note-schema.md's "Project Note Body
   Structure" — every note created here has `para: projects`, so it always gets this shape,
   never the generic Purpose-line body). Derive `title` from the slug per the rule above.
   Put the item text (minus all routing tags — `@workspace`, `#slug`, `!N`, `due:`, `dep:`)
   into the `Starting State` section as the raw capture — it's the seed description of why
   this note exists, not yet a stated goal or plan. Leave `Goal`, `Approach`, and
   `Problems & Dependencies` as bare placeholder comments for the user to fill in later.
   Set `priority` from `!N` if present (default `6` otherwise). Set `due` from `due:` if
   present (omit the field otherwise). Set `depends-on` from any `dep:` entries (empty list
   `[]` if none), and mirror each one into the `> [!info]- Dependencies` body callout as
   `[[<path>|<derived label>]]`, deriving the label the same way the title is derived.
   Confirm creation to user, then remove the item from INBOX.

### Case B — `@workspace` only (no `#project-slug`)

1. Confirm the workspace folder exists. If not, treat as Case C.
2. Derive a kebab-case slug from the item text (strip all routing tags, slugify remaining text).
3. **Before creating anything, check for a collision** — a bare `@workspace` capture is
   often about something that already has a note, even though there's no explicit
   `#project-slug` pointing at it (e.g. `Bootdev @Study` landing on an already-existing
   `Study/projects/bootdev.md`). If the derived slug matches an existing file, or an
   existing note in `<workspace>/projects/` is obviously the same topic, treat it exactly
   like Case A's "file exists" branch — ask **open it** vs. **leave in INBOX** — rather than
   silently creating a duplicate note.
4. **No collision** → create a new project note at `<workspace>/projects/<slug>.md` using
   standard frontmatter and the four-section project body, applying `priority`/`due`/`depends-on`
   from `!N`/`due:`/`dep:` tags and the `Starting State` placement exactly as in Case A.
5. Confirm to user, then remove the item from INBOX.

### Case C — no routing tags

1. Show the user the item text.
2. Ask: "Where should this go?"
3. User responses:
   - **"whiteboard"** → append the item (with today's date appended) to the `## Whiteboard` section in INBOX.md. Remove from main list.
   - **"@workspace #project-slug"** → re-route as Case A.
   - **"@workspace"** → re-route as Case B.
   - **"leave"** → leave the item in the main list untouched.

### Optional pattern: route the item AND add an inline `TODO-N:` marker

Sometimes the user's routing answer includes an explicit `TODO-N:` (with or without a
message after the colon) — e.g. "route this to `@van_chores #electric_cupboard_fans
TODO-3`". That means: file it normally (Case A/B, `Starting State` gets the raw item
text as usual), **and additionally** add a `## Notes` section to the note with a
`- [ ] TODO-N: <message>` checkbox line, per `note-schema.md`'s Inline TODO Markers
convention. The marker is *in addition to* the `Starting State` text, not a replacement
for it — don't drop one for the other.

### Optional pattern: consolidating scattered TODOs into a new note

Occasionally an inbox item implies gathering existing, related open items scattered
across multiple already-filed notes into one new note, rather than describing something
standalone (e.g. "GAS lib framework upgrades, document so far" turning into pulling every
README/documentation `TODO-N:` out of several library notes into one new
`documentation.md`). This is a bigger, multi-file operation — handle it as its own step,
not as routine Case A/B filing:

1. Identify the candidate existing items/notes that plausibly belong in scope.
2. Get **explicit confirmation** from the user on exactly which ones are in scope before
   touching anything — a multi-select list of candidates works well here.
3. Move (don't duplicate) the confirmed content: delete it from the source note, bump
   that note's `updated:` date, and add it to the new consolidated note.
4. If a source note ends up with no remaining live content of its own (e.g. its entire
   purpose was superseded), retire it to `<workspace>/archive/` (`para: archive`,
   `status: complete` or `cancelled`, per `note-schema.md`'s archive convention) with a
   one-line pointer back to the new note, rather than deleting it outright.

---

## After processing

- Rewrite INBOX.md: remove all successfully filed items from the main list.
- Leave skipped ("leave") items in place.
- Leave the `## Whiteboard` section completely untouched — never modify it during processing.
- Preserve the header comment block at the top of the file.
- Report a summary: how many items filed, how many left, how many sent to whiteboard.

---

## Rules

- Never process items inside `## Whiteboard` — that section is manual-only.
- Never delete a whiteboard item during processing.
- If the vault root cannot be determined, stop and tell the user.
- New notes created during processing must follow the schema in `.claude/skills/note-schema.md`.
- Do not batch-process silently — always confirm with the user before creating or modifying a note.
