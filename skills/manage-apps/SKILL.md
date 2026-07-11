---
name: manage-apps
description: Browse the 88eggs app catalog (App Store) and manage a project's installed apps -- list what's available, install an app into a project, list/show installed apps, uninstall, and list an app's pages -- by driving the 88eggs CLI. Use when the user asks to see, install, uninstall, or inspect 88eggs apps for a project.
---

# Manage Apps

Browses the 88eggs **App Store** and manages a project's **installed apps**
by driving the [`88eggs` CLI](https://github.com/ptk-studio/88eggs-cli),
the same way `vercel-labs`'s deploy skill drives the `vercel` CLI. This
skill never handles a token directly, and never calls `88eggs-backend`
with `curl`/`fetch` itself; `88eggs` owns its own login and every API call.

An **app** is something installed *into a project* (from the catalog of
listings); installing it creates a new installed instance you can then
manage. Apps can carry public **pages** served at their own URLs.

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
working around it — don't sudo, don't try an alternate package manager,
don't fall back to cloning source.

## Step 2: Check sign-in state

```bash
88eggs whoami
```

- Prints the signed-in email and exits `0` → already signed in, go to
  Step 3.
- Prints `Not signed in. Run \`88eggs login\`.` and exits `1` → not
  signed in.

If not signed in, **tell the user and ask before proceeding** — `88eggs
login` opens their browser for a real Google sign-in:

```
You're not signed in to 88eggs. I can run `88eggs login`, which will
open your browser to sign in with Google. Want me to proceed?
```

Once they confirm, run `88eggs login` and wait for it to complete before
continuing.

## Step 3: Do what the user actually asked

Apps are always scoped to a project when installing/listing. If the user
names a project, use it; if not and the action needs a `--project`, run
`88eggs projects list` first and ask which one (or use the only one, if
they have just one — see the `list-projects` skill).

- **Browse the catalog**: `88eggs apps catalog` — one line per available
  app, `<slug> -- <name> -- <description>`. The `<slug>` is what
  `install` takes. If it's empty, tell the user there's nothing to
  install yet, don't invent an app.
- **Install into a project**: `88eggs apps install <slug> --project
  <projectId>` — installs the catalog app named by `<slug>`, taking the
  listing's own name (pass `--name <name>` only if the user wants a
  custom name). Prints `Installed "<name>" (<appId>) into project
  <projectId>.` — note the app id. Installing the same catalog app again
  after uninstalling it makes a fresh install; only **409 "already
  installed"** if it's currently active (see Errors).
- **List a project's installed apps**: `88eggs apps list --project
  <projectId>` — `<appId> -- <name> -- installed <timestamp>` per active
  install.
- **Show one app**: `88eggs apps show <appId>` — its name, project,
  catalog listing, status, and created time.
- **Uninstall**: `88eggs apps uninstall <appId>` — archives it (its pages
  are kept and it can be reinstalled). **Confirm with the user before
  running this** — it removes the app from the project's active list.
- **List an app's pages**: `88eggs apps pages <appId>` (add `--archived`
  to include archived pages) — `<pageId> -- <title> -- slug: <slug>` per
  page.

Present results as a short summary, not raw command output verbatim —
e.g. "Acme Launch has 2 apps installed; the newest is Landing Page App
from yesterday," not a pasted list.

### Errors

- Any non-zero exit with an `Error: ...` line on stderr — surface that
  message directly.
- `Error: This app is already installed in the project` — the catalog app
  is currently active in that project; there's nothing to do (or the user
  meant a different project). Don't retry.
- `Error: No app in the catalog with slug "..."` — the slug doesn't match
  the catalog; re-check `88eggs apps catalog`, don't guess a close
  spelling.
- `Error: Not found` on an app/project-scoped command usually means it
  doesn't exist or the caller can't access its project — indistinguishable
  by design (`88eggs-backend`'s "404 not 403" convention), so don't imply
  one over the other.
- A `401`/session-related failure will say to run `88eggs login` again;
  don't attempt to fix this yourself, tell the user.
