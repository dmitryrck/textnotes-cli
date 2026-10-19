# textnotes-cli

`textnotes-cli` is a CLI for your second brain, it is backed by CouchDB with
Nouveau full-text search.

## Quickstart

(1) Download from the releases page, place it in your path.

(2) Add your credentials to `~/.netrc`, example:

```
machine 127.0.0.1
  login remoteuser
  password secret
```

After that index your CouchDB:

```bash
textnotes --server-url 'http://127.0.0.1:5984/my-db' index
```

If you don't want to always use `--server-url` you can export the environment
variable `TEXTNOTES_URL` with the same content:

```
export TEXTNOTES_URL=http://127.0.0.1:5984/my-db
```

## Global flags

`--format type`, output format, `type` can be `human` (default) or `json`

This flag is ignored by `new` or `edit`, as these are interactive commands and
not intended for scripting.

`--server-url https://example.com/my-db` address with the database for your
CouchDB instance.

If you don't want to always provide `--server-url` you can export the
environment variable `TEXTNOTES_URL` with the same content.

## Commands

Besides the commands below, `version` and `help` is also available.

Each subcommand also accepts `--help`.

### list - List all notes

```
textnotes list
textnotes list --page 1
textnotes --format json list
```

### search - Search notes using FTS

You need to have Nouveau in your CouchDB server, otherwise this will not work.

```
textnotes search something cool
```

According to Nouveau, the order of the words does not matter.

### read - Read one note

Read one note and output as frontmatter (default) or JSON.

```
textnotes read --id <id>
textnotes read --title "Hello world"
```

### create - Create one note

Create one note with given attributes, `--title` is required:

```
textnotes create --title "My Note" \
  --content "# Hello" \
  --content-type "text/markdown" \
  --categories "single-word-category [[multi word category]]" \
  --url "https://example.com" \
  --fields-entry-klass "Article" \
  --fields-author "Some Author"
```

Options:

* `--title`, title (quote if using multiple words or it has spaces/special chars)
* `--content`, body (markdown by default)
* `--content-type`, default `text/markdown` (saved as `content_type`)
* `--categories`, space-separated list of categories use `[[multi word]]` for categories with space
* `--url`, URL to attach, for your reference only
* `--fields-<key> <value>`, add some metadata, example: `--fields-icon "rocket-icon"`, `--fields-score 5`

### update - Fully replace a note

Find a note by id and update it.

`--id` and `--title` are required. Omitted fields are **cleared**, example:

```
textnotes update --id <id> --title "New Title" \
  --content "replaced" --content-type "text/markdown"
```

Accept the same parameters as `create`.

### patch - Partial update

Find a note by id and update the parameters given in the CLI.

Only `--id` is required. Omitted fields are kept as is:

```
textnotes patch --id <id> --content "replaced"
```

Accept the same parameters as `create` and `update`.

### delete - Delete note by id

```bash
textnotes delete --id <id>
```

Cannot be undone.
