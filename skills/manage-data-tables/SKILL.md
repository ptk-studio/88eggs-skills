---
name: manage-data-tables
description: Create and manage 88eggs data tables and their rows -- list tables, create one with columns, edit its schema, list/add/update/delete rows, archive/restore, and publish a table's schema as a template -- by driving the 88eggs CLI. Use when the user asks to create, list, edit, or manage an 88eggs data table, its columns, or its rows.
---

# Manage Data Tables

Creates and manages 88eggs **data tables** (a project's user-defined tables
that pipelines read and write between runs) and their **rows**, by driving the
[`88eggs` CLI](https://github.com/ptk-studio/88eggs-cli), the same way
`vercel-labs`'s deploy skill drives the `vercel` CLI. This skill never handles
a token directly, and never calls `88eggs-backend` with `curl`/`fetch` itself;
`88eggs` owns its own login and every API call.

A **data table** is scoped to one project. Its **columns** are advisory display
hints (every cell is stored as a string) — a row can carry extra keys not in
the schema. Every table has a built-in `Id` and a built-in `status`, so you
never add those as columns; `status` is set per row via `--status`, not as a
`--cell`.

## Step 1: Check the CLI is installed

```bash
command -v 88eggs
```

If missing, install it — it's a published npm package, a one-line global
install with no build step:

```bash
npm install -g 88eggs-cli
```

If this fails (no npm, permission error, etc.), tell the user rather than
working around it — don't sudo, don't try an alternate package manager, don't
fall back to cloning source.

## Step 2: Check sign-in state

```bash
88eggs whoami
```

- Prints the signed-in email and exits `0` → already signed in, go to Step 3.
- Prints `Not signed in. Run \`88eggs login\`.` and exits `1` → not signed in.

If not signed in, **tell the user and ask before proceeding** — `88eggs login`
opens their browser for a real Google sign-in:

```
You're not signed in to 88eggs. I can run `88eggs login`, which will open
your browser to sign in with Google. Want me to proceed?
```

Once they confirm, run `88eggs login` and wait for it to complete before
continuing.

## Step 3: Do what the user actually asked

Tables are scoped to a project. If the user names a project, use it; if not and
the action needs a `--project`, run `88eggs projects list` first and ask which
one (or use the only one, if they have just one — see the `list-projects`
skill). `create` may omit `--project` (defaults to their oldest project) — say
which project you used.

- **List tables**: `88eggs data-tables list [--project <projectId>]` (add
  `--archived` for archived tables) — `<tableId> -- <name> -- [col keys] --
  project <projectId>` per table.
- **Show a table's schema**: `88eggs data-tables show <tableId>` — name,
  project, status, and each column's `key -- label (type)`.
- **Create a table**: `88eggs data-tables create <name> [--project
  <projectId>] --column key:type[:label] ...`. `type` is one of
  `text | textarea | number | boolean | asset` (`textarea` = long multi-line
  text; `asset` = the cell is an asset id). Repeat `--column` per column, in
  order. Don't add `Id`/`status` columns — they're built in. Prints the new
  table id. Example:
  ```bash
  88eggs data-tables create "Story1" \
    --column story_collection:text \
    --column story_name:text \
    --column story_topic:textarea \
    --column story_num_scenes:number
  ```
- **Edit a table**: `88eggs data-tables update <tableId>` with any of
  `--name <name>`, `--column ...` (repeat to **replace** the whole column
  list — additive edits never touch existing rows), `--status-option <value>`
  (repeat to restrict the row editor's status dropdown), `--archive` /
  `--restore`. **Confirm before `--archive`** — it hides the table from default
  listings (rows and pipeline steps referencing it by id keep working).
- **Publish the schema**: `88eggs data-tables publish <tableId> [--name
  <name>]` — shares the columns + name to the Templates catalog as a
  clonable `data-table` template. Rows are never shared. Confirm first (it's
  outward-facing — any user can clone the schema).

Rows:

- **List rows**: `88eggs data-tables rows list <tableId>` (newest first;
  `--column <key> --value <value>` to filter — use `--column status` for the
  status column, `--value ""` matches rows with no status). `--page`/`--limit`
  paginate. Prints `<rowId> -- [status] -- key=value ...`.
- **Show one row**: `88eggs data-tables rows show <tableId> <rowId>`.
- **Add a row**: `88eggs data-tables rows add <tableId> --cell key=value ...
  [--status <status>]`. Cells are the column values; `--status` sets the
  built-in status column (not a `--cell`). Example:
  ```bash
  88eggs data-tables rows add <tableId> \
    --cell story_collection=HoneyStorm --cell story_name="The First Swarm" \
    --status STORY_CREATED
  ```
- **Update a row**: `88eggs data-tables rows update <tableId> <rowId> --cell
  key=value ... [--status <status>]` — merges the given cells; omitted cells
  are left alone. `--status ""` clears the status.
- **Delete a row**: `88eggs data-tables rows delete <tableId> <rowId>`.
  **Confirm with the user before running this** — it permanently removes the
  row.

Present results as a short summary, not raw command output verbatim — e.g.
"Story1 has 3 rows; the newest is 'The First Swarm' (STORY_CREATED)," not a
pasted list.

### Errors

- Any non-zero exit with an `Error: ...` line on stderr — surface that message
  directly.
- `Error: Invalid --column "..."` / `Invalid column type "..."` — the spec
  isn't `key:type[:label]` or the type isn't one of the five; fix and retry.
- `Error: Nothing to update` — an `update` with no `--name`/`--column`/
  `--status-option`/`--archive`/`--restore` (table) or no `--cell`/`--status`
  (row); nothing was sent.
- `Error: Not found` on a table/row command usually means it doesn't exist or
  the caller can't access its project — indistinguishable by design
  (`88eggs-backend`'s "404 not 403" convention), so don't imply one over the
  other.
- A `409` on a row insert means the table hit its 100,000-row cap (a
  runaway-loop backstop) — tell the user, don't retry.
- A `401`/session-related failure will say to run `88eggs login` again; don't
  attempt to fix this yourself, tell the user.
