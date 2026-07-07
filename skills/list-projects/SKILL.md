---
name: list-projects
description: List the signed-in user's 88eggs projects (owned and team-shared) by driving the 88eggs CLI. Use when the user asks to see, list, or check their 88eggs projects.
---

# List Projects

Lists the caller's own 88eggs projects — owned and shared with any team
they belong to — by driving the [`88eggs` CLI](https://github.com/ptk-studio/88eggs-cli),
the same way `vercel-labs`'s deploy skill drives the `vercel` CLI. This
skill never handles a token directly; `88eggs` owns its own login.

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

- Prints the signed-in email and exits `0` → already signed in, go to
  Step 3.
- Prints `Not signed in. Run \`88eggs login\`.` and exits `1` → not
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

## Step 3: List projects

```bash
88eggs projects list
```

Optionally scope it: `88eggs projects list --scope mine` (owned only)
or `--scope shared` (shared with a team only); omit for everything the
caller can see (the default).

Each line is `<name> -- owner: <name> -- updated <timestamp>` (or `--
shared with <team name> --` when shared). Present these to the user as
a short summary, not the raw command output verbatim.

### Errors

- Any non-zero exit with an `Error: ...` line on stderr — surface that
  message directly. A `401`/session-related failure will say to run
  `88eggs login` again; don't attempt to fix this yourself, tell the
  user.
