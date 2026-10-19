---
name: textnotes
description: Tools for searching and creating notes in my second brain.
---

# Textnotes

CLI for the user's second brain (notes database). Binary is `textnotes` (also `textnotes-cli`).

Use the `bash` tool to invoke. Global flags **must come before** the subcommand:

- `--server-url <URL>` — server URL, e.g. `https://example.com/database` (without trailing slash)
- `--format <FORMAT>` — `human` (default) or `json` (machine-readable, more metadata). Prefer `json` for scripting.

```bash
textnotes --format json list
textnotes --server-url https://example.com/database --format json search foo
```

Available subcommands: `help`, `list`, `search`, `read`, `create`, `update`, `patch`, `delete`.

## `textnotes list`

List all notes. No arguments.

```bash
textnotes list
textnotes --format json list
```

- `human`: `+ <Title> (id: <id>)` per line
- `json`: `{"docs": [{_id, _rev, title, content, content_type, categories, url, fields, klass, ...}]}`

## `textnotes search <word> [word ...]`

Search notes by **title** (one or more positional words).

```bash
textnotes search rocket
textnotes --format json search test cli
textnotes --format json search "multi word"
```

- `human`: `+ <Title> (id: <id>)` per hit
- `json`: `{"docs": [<full note objects>]}`

## `textnotes read`

Read a single note by ID or by exact title (one of the two required):

```bash
textnotes read --id han-46d7ac0a38a94c0595621da014809711
textnotes read --title "rocket-icon"
textnotes --format json read --id han-46d7ac0a38a94c559552cebfeed12211
textnotes --format json read --title "Test from the cli"
```

Options:
- `--id <ID>` — note ID (e.g. `han-...`)
- `--title <TITLE>` — exact title match

- `human`: `---` frontmatter (`_id`, `title`, etc.) + markdown body
- `json`: single full note object `{_id, title, content, content_type, ...}`

## `textnotes create`

Create a note. `--title` is the only mandatory flag:

```bash
textnotes create --title "My Note" --content "# Hello" --content-type "text/markdown" --categories "single-word [[multi word]]" --url "https://example.com"
textnotes --format json create --title "My Note" --content "body"
```

Options:
- `--title <TITLE>` — title
- `--content <CONTENT>` — body (markdown by default)
- `--content-type <TYPE>` — default `text/markdown`, e.g. `image/svg+xml`
- `--categories <CATEGORIES>` — space-separated, use `[[multi word]]` for categories with spaces
- `--url <URL>` — URL to attach
- `--fields-<key> <VALUE>` — extra metadata stored in `fields` (repeatable), e.g. `--fields-icon "rocket-icon"` or `--fields-custom-key "myvalue"` — as hinted in `textnotes help`

- `human`: `+ Created: <Title> (id: <id>)`
- `json`: `{"id": "...", "ok": true, "rev": "..."}` (verify with `read`/`list` to see `fields`)

## `textnotes update`

**Fully replace** a note. `--id` is mandatory; omitted fields are cleared.

```bash
textnotes update --id han-06d4f4b49d264c0dad300f500cbdd713 --title "New Title" --content "replaced" --categories "" --url "" --content-type "text/markdown"
textnotes --format json update --id han-xxx --title "New" --content "new body"
```

Options: `--id <ID>` (required), plus `--title` (required), `--categories`, `--url`, `--content-type`, `--content`, and `--fields-<key> <VALUE>` (same as `create`; extra metadata via `--fields-custom-key CUSTOM_VALUE`).

Returns: `{"id": "...", "ok": true, "rev": "..."}` in json, or `+ Updated ...` in 'human' output mode.

## `textnotes patch`

**Partial update** — only provided fields are changed, others preserved. `--id` mandatory.

```bash
textnotes patch --id han-xxx --content "patched only this field"
textnotes patch --id han-xxx --title "Renamed"
textnotes --format json patch --id han-xxx --content "new content"
```

Options: same as `update` (`--id` required, others optional, plus `--fields-<key> <VALUE>` for extra metadata — e.g. `--fields-icon "new"` — as hinted in `textnotes help`).

Returns: `{"id": "...", "ok": true, "rev": "..."}` in json.

Use `patch` for safe edits (only provided fields changed); use `update` when you want to fully replace the note (omitted fields cleared).

## `textnotes delete`

Delete a note by ID (required):

```bash
textnotes delete --id han-06d4f4b49d264c0dad300f500cbdd713
textnotes --format json delete --id han-xxx
```

- `human`: `+ Deleted: id: <id>`
- `json`: `{"id": "...", "ok": true, "rev": "..."}`

## `textnotes help`

Shows top-level help. Note the hints for extra metadata:
- `create` — `create a note (extra metadata can be added with --fields-custom-key CUSTOM_VALUE)`
- `update` — `update a note by fully replacing it (extra metadata can be added with --fields-custom-key CUSTOM_VALUE)`
- `patch` — `update a note by changing only the fields that are provided (extra metadata can also be changed with --fields-custom-key CUSTOM_VALUE)`

```bash
textnotes help
textnotes --help
textnotes list --help
textnotes search --help
textnotes create --help
textnotes update --help
textnotes patch --help
```

## Tips

- Quote titles/content with spaces: `--title "This is a test"` 
- For scripting always use `--format json` **before** subcommand: `textnotes --format json search example | python3 -m json.tool`
- JSON for `list`/`search` is `{"docs": [...]}` — not one-object-per-line.
- Set content type with `--content-type` (default `text/markdown`); it is saved as `content_type` in JSON.
- Verify writes with `read`: `textnotes --format json read --id <id>`
