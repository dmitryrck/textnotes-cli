# textnotes-cli

`textnotes-cli` is a CLI for your second brain, it is backed by CouchDB with
Nouveau full-text search.

## Quickstart

(1) Download from the releases page, place it in your path.

(2) Add your credentials to `~/.netrc`, example:

```
machine example.com
  login remoteuser
  password secret
```

Replace `example.com` with the correct domain or IP address. In case of an Ip
address don't add the protocol or port, only the IP address.

(3) After that index your CouchDB:

```
textnotes --server-url 'https://example.com/my-db' index
```

Note:

If you don't want to always use `--server-url` you can export the environment
variable `TEXTNOTES_URL` with the same content:

```
export TEXTNOTES_URL=https://example.com/my-db
```

(4) Use the subcommand `new` to open your editor with an empty note. Make sure
to have `VISUAL` or `EDITOR`.

```
textnotes new
```

Save and exit your editor. It will output:

```
Created:
 + Hello World (id: han-914814d805194b1baf00273602a71f22)
```

You can edit with:

```
./textnotes edit --id han-914814d805194b1baf00273602a71f22
```

You can delete with:

```
./textnotes delete --id han-914814d805194b1baf00273602a71f22
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
textnotes create \
  --title "My Note" \
  --content "# Hello" \
  --content-type "text/markdown" \
  --categories "single-word-category [[multi word category]]" \
  --url "https://example.com" \
  --fields-entry-klass "Article" \
  --fields-author "Some Author"
```

Options:

* `--title`, title (quote if using multiple words or it has special chars)
* `--content`, body of the note
* `--content-type`, default `text/markdown` (saved in CouchDB as `content_type`)
* `--categories`, space-separated list of categories use `[[multi word]]` for categories with space, example: `simple-category [[multi word]]`
* `--url`, URL to attach, for your reference only
* `--fields-<key> <value>`, add some metadata, example: `--fields-icon "rocket-icon"`, `--fields-score 5`

### update - Fully replace a note

Find a note by id and update it.

`--id` and `--title` are required. Omitted fields are **cleared**, example:

```
textnotes update \
  --id <id> \
  --title "New Title" \
  --content "replaced" --content-type "text/markdown"
```

Accepts the same parameters as `create`.

### patch - Partial update

Find a note by id and update the parameters given in the CLI.

Only `--id` is required. Omitted fields are kept as is:

```
textnotes patch --id <id> --content "replaced"
```

Accepts the same parameters as `create` and `update`.

### delete - Delete note by id

```
textnotes delete --id <id>
```

Cannot be undone.
