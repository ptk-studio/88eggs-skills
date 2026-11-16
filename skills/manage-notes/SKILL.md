---
name: manage-notes
description: Create and manage 88eggs notes -- a per-project markdown wiki -- by driving the 88eggs CLI. List a project's notes as a flat list or a tree (children under a parent), read/create/edit a note's markdown, nest notes under a parent, cross-link notes with [[path]] and embed project assets with [[asset:id]]/![[asset:id]], and archive/restore. Use when the user asks to create, list, read, edit, organize, link, or archive an 88eggs note.
---

# Manage Notes

Creates and manages 88eggs **notes** (a project's markdown documents that form a
small internal wiki) by driving the
[`88eggs` CLI](https://github.com/ptk-studio/88eggs-cli), the same way
`vercel-labs`'s deploy skill drives the `vercel` CLI. This skill never handles a
token directly and never calls `88eggs-backend` with `curl`/`fetch` itself;
`88eggs` owns its own login and every API call.

A **note** is scoped to one project and has:
- a **path** — a unique slug per project (`^[a-zA-Z0-9._-]+$`: letters any
  case, digits, `.`, `-`, `_`; no slashes or spaces — so `README.md` works).
  This is the note's stable key.
- a **name** — the display title.
- **content** — markdown.
- a **parent_path** — the path of its parent note (empty = top-level). Notes
  form a tree: a note's children are the notes whose `parent_path` is its path.

Inside a note's markdown, `[[<path>]]` links to another note in the same
project, and `[[asset:<id>]]` links (or `![[asset:<id>]]` embeds) one of the
project's assets. Notes are archived, never hard-deleted.

## Step 1: Check the CLI is installed

```bash
command -v 88eggs
```

If missing, install it — it's a published npm package, a one-line global install
with no build step:

```bash
npm install -g 88eggs-cli
```

(Notes commands need `88eggs-cli` ≥ 0.9.0, and sign-in needs ≥ 0.10.0; if `88eggs notes --help` errors on an
older global install, `npm install -g 88eggs-cli@latest`.)

If install fails (no npm, permission error, etc.), tell the user rather than
working around it — don't sudo, don't try an alternate package manager, don't
fall back to cloning source.

## Step 2: Check sign-in state

```bash
88eggs whoami
```

- Prints the signed-in email and exits `0` → already signed in, go to Step 3.
- Prints `Not signed in. Run \`88eggs login\`.` and exits `1` → not signed in.

If not signed in, **tell the user; don't run `88eggs login` yourself.** Since
88eggs-cli 0.10.0, `88eggs login` asks for their 88eggs username (or email)
and password in a terminal, and only they should type their password:

```
You're not signed in to 88eggs. Please run `88eggs login` in a terminal
(it asks for your 88eggs username or email and password), then tell me
when it says "Signed in as ...".
```

Wait for them to confirm, re-run `88eggs whoami` to check, then continue.

## Step 3: Do what the user actually asked

Notes are scoped to a project, and every notes command needs a `--project`
(except `show`/`update`, which take a note id). If the user names a project, use
it; otherwise run `88eggs projects list` and ask which one (or use the only one,
if they have just one — see the `list-projects` skill).

- **List notes**: `88eggs notes list --project <projectId>` — one
  `<noteId> -- <name> -- <path> (under <parent>, +children, archived)` per note.
  By default this is a flat list; to walk the tree:
  - `--roots` lists only top-level notes,
  - `--parent <path>` lists the direct children of that note.
  Add `--archived` (archived only) or `--all` (both), `--q <text>` to search
  name/path, `--page`/`--limit` to paginate. A `+children` tag means the note
  has children you can list with `--parent <its path>`.
- **Read a note**: `88eggs notes show <noteId>` (metadata + full markdown), or
  by its slug: `88eggs notes by-path --project <projectId> --path <slug>`.
- **Create a note**: `88eggs notes create "<name>" --project <projectId> --path
  <slug> [--content "<markdown>" | --content-file <file.md>] [--parent <path>]`.
  `--path` must be a valid slug (`^[a-zA-Z0-9._-]+$`) and unique among the project's
  active notes. Prefer `--content-file` for anything longer than a line.
  `--parent` nests it under another note (omit for top-level). Example:
  ```bash
  88eggs notes create "Brand Voice" --project <projectId> \
    --path brand-voice --content-file ./brand-voice.md --parent guides
  ```
- **Edit a note**: `88eggs notes update <noteId>` with any of `--name`,
  `--path` (renaming re-links children automatically), `--content` /
  `--content-file` (replace the body), `--parent <path>` or `--top-level`
  (re-home in the tree), `--archive` / `--restore`.
- **Archive a note**: `88eggs notes update <noteId> --archive`. **Confirm before
  archiving** — it cascades to the note's descendants (the whole subtree). It is
  reversible with `--restore` (which restores just that note).

### Cross-links and asset embeds (edit the markdown `content`)

- Link another note: put `[[<path>]]` (or `[[<path>|Label]]`) in the content —
  it resolves to that note's page in the app.
- Link a project asset: `[[asset:<assetId>]]`; embed it inline (image/video/
  audio) with `![[asset:<assetId>]]`. Find asset ids with the `manage-assets`
  skill / `88eggs assets list --project <projectId>`.

Present results as a short summary, not raw command output — e.g. "The project
has 4 top-level notes; 'Guides' has 3 children," not a pasted list.

### Errors

- Any non-zero exit with an `Error: ...` line on stderr — surface that message
  directly.
- `Error: Not found` on a note command usually means it doesn't exist or the
  caller can't access its project — indistinguishable by design
  (`88eggs-backend`'s "404 not 403" convention), so don't imply one over the
  other.
- A `400` on create/update about the path means the slug isn't `^[a-zA-Z0-9._-]+$`,
  or a `--parent`/`--top-level` target isn't a real note path in the project
  (or would create a cycle) — fix and retry.
- A `409` on create/update means another **active** note already holds that
  `--path` in the project — pick a different slug (or archive/rename the other
  one first).
- `Error: nothing to update` — an `update` with no changing flags; nothing was
  sent.
- `Error: pass only one of --content or --content-file` — pick one source for
  the body.
- A `401`/session failure will say to run `88eggs login` again; don't try to fix
  it yourself, tell the user.
