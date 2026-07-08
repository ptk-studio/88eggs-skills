---
name: manage-assets
description: Browse and organize 88eggs assets (images/videos) — list a project's assets, view one item's signed URL, filter/manage tags, like/unlike, move between projects — by driving the 88eggs CLI. Use when the user asks to see, find, tag, like, or move their 88eggs assets, images, or videos.
---

# Manage Assets

Browses and organizes the caller's own 88eggs assets by driving the
[`88eggs` CLI](https://github.com/ptk-studio/88eggs-cli), the same way
`vercel-labs`'s deploy skill drives the `vercel` CLI. This skill never
handles a token directly; `88eggs` owns its own login.

## Step 1: Check the CLI is installed

```bash
command -v 88eggs
```

If missing, install it — it's a published npm package, a one-line global
install with no build step, same as how a missing `vercel` CLI gets
installed automatically:

```bash
npm install -g 88eggs-cli
```

If this fails (no npm, permission error, etc.), tell the user rather
than working around it — don't sudo, don't try an alternate package
manager, don't fall back to cloning source.

## Step 2: Check sign-in state

```bash
88eggs whoami
```

- Prints the signed-in email and exits `0` -> already signed in, go to
  Step 3.
- Prints `Not signed in. Run \`88eggs login\`.` and exits `1` -> not
  signed in.

If not signed in, **tell the user and ask before proceeding** — `88eggs
login` opens their browser for a real Google sign-in, which needs their
own interaction:

```
You're not signed in to 88eggs. I can run `88eggs login`, which will
open your browser to sign in with Google. Want me to proceed?
```

Once they confirm, run `88eggs login` and wait for it to complete (it
prints "Signed in as `<email>`." on success) before continuing.

## Step 3: Do what the user actually asked

Assets are always scoped to a project. If the user names one, use it; if
not and the action needs a `--project`/`<projectId>`, run `88eggs
projects list` first and ask which project (or use the only one, if
they have just one).

- **List a project's assets**: `88eggs assets list --project <projectId>`
  (add `--tag <tag>` to filter by tag, `--run-name <name>` to filter by
  the run that produced it — a partial, case-insensitive match, useful
  when the user describes what they're looking for by what they named
  the run rather than by a tag — `--page`/`--limit` to paginate). Each
  line is `<id> -- <type> -- <tags> -- run "<name>" -- created
  <timestamp>` (the `run "..."` segment only appears for an asset that has
  one; `-- liked` appended when the caller likes it), followed by a
  `-- page N (limit L, total T)` summary line.
- **See what tags exist**: `88eggs assets tags` (every accessible
  project) or `88eggs assets tags --project <projectId>` (one project) —
  useful before filtering a list by tag.
- **View one item / get a shareable link**: `88eggs assets show
  <assetId>` — prints its project, type, tags, producing run's name (if
  it has one), liked state, and a signed URL (valid 24h; don't cache it
  beyond that).
- **See what's liked**: `88eggs assets liked` (same pagination flags as
  list).
- **Like / unlike**: `88eggs assets like <assetId>` / `88eggs assets
  unlike <assetId>`.
- **Add / remove a tag**: `88eggs assets tag add <assetId> <tag>` /
  `88eggs assets tag remove <assetId> <tag>`.
- **Move to a different project**: `88eggs assets move <assetId>
  <projectId>` — confirm the destination project with the user before
  running this; it changes who else can see the item (anyone with
  access to the new project, not just the old one).

Present results to the user as a short summary, not raw command output
verbatim — e.g. "You have 12 images tagged `hero-shot` in Acme
Launch, most recent from March 3rd," not a pasted list.

### Errors

- Any non-zero exit with an `Error: ...` line on stderr — surface that
  message directly.
- `Error: Not found` on an item/project-scoped command usually means
  either it doesn't exist or the caller doesn't have access to its
  project — these are indistinguishable by design (see
  `88eggs-backend`'s "404 not 403" convention), so don't imply one over
  the other to the user.
- A `401`/session-related failure will say to run `88eggs login` again;
  don't attempt to fix this yourself, tell the user.
