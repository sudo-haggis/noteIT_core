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
The original item text (minus tags) goes in the note body's Purpose line instead, so
detail isn't lost even though the title stays short.

---

## Processing logic

Work through each item in the main list (everything above `## Whiteboard`) one at a time.

### Case A — `@workspace #project-slug` present

1. Normalize `#project-slug` to kebab-case, then resolve `<workspace>/projects/<project-slug>.md`.
2. **File exists** → Tell the user the item and the existing note path. Ask:
   - **Open it** — open the note for the user to edit manually, then remove the item from INBOX.
   - **Leave in INBOX** — skip this item and leave it in the list untouched.
3. **File does not exist** → Create a new project note using the standard frontmatter template
   (see note-schema.md). Derive `title` from the slug per the rule above. Put the item text
   (minus all routing tags — `@workspace`, `#slug`, `!N`, `due:`, `dep:`) into the Purpose
   line in the body. Set `priority` from `!N` if present (default `6` otherwise). Set `due`
   from `due:` if present (omit the field otherwise). Set `depends-on` from any `dep:`
   entries (empty list `[]` if none), and mirror each one into the `> [!info]- Dependencies`
   body callout as `[[<path>|<derived label>]]`, deriving the label the same way the title
   is derived. Confirm creation to user, then remove the item from INBOX.

### Case B — `@workspace` only (no `#project-slug`)

1. Confirm the workspace folder exists. If not, treat as Case C.
2. Derive a kebab-case slug from the item text (strip all routing tags, slugify remaining text).
3. Create a new project note at `<workspace>/projects/<slug>.md` using standard frontmatter,
   applying `priority`/`due`/`depends-on` from `!N`/`due:`/`dep:` tags exactly as in Case A.
4. Confirm to user, then remove the item from INBOX.

### Case C — no routing tags

1. Show the user the item text.
2. Ask: "Where should this go?"
3. User responses:
   - **"whiteboard"** → append the item (with today's date appended) to the `## Whiteboard` section in INBOX.md. Remove from main list.
   - **"@workspace #project-slug"** → re-route as Case A.
   - **"@workspace"** → re-route as Case B.
   - **"leave"** → leave the item in the main list untouched.

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
