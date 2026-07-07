---
name: list-projects
description: List the signed-in user's Rainbow projects (owned and team-shared) by driving the rainbow CLI. Use when the user asks to see, list, or check their Rainbow projects.
---

# List Projects

Lists the caller's own Rainbow projects — owned and shared with any team
they belong to — by driving the [`rainbow` CLI](https://github.com/ptk-studio/rainbow-cli),
the same way `vercel-labs`'s deploy skill drives the `vercel` CLI. This
skill never handles a token directly; `rainbow` owns its own login.

## Step 1: Check the CLI is installed

```bash
command -v rainbow
```

If missing, tell the user to install it from
[ptk-studio/rainbow-cli](https://github.com/ptk-studio/rainbow-cli)
(`pnpm install && pnpm build && npm link`) rather than attempting to
install it yourself.

## Step 2: Check sign-in state

```bash
rainbow whoami
```

- Prints the signed-in email and exits `0` → already signed in, go to
  Step 3.
- Prints `Not signed in. Run \`rainbow login\`.` and exits `1` → not
  signed in.

If not signed in, **tell the user and ask before proceeding** — `rainbow
login` opens their browser for a real Google sign-in, which needs their
own interaction:

```
You're not signed in to Rainbow. I can run `rainbow login`, which will
open your browser to sign in with Google. Want me to proceed?
```

Once they confirm, run `rainbow login` and wait for it to complete (it
prints "Signed in as `<email>`." on success) before continuing.

## Step 3: List projects

```bash
rainbow projects list
```

Optionally scope it: `rainbow projects list --scope mine` (owned only)
or `--scope shared` (shared with a team only); omit for everything the
caller can see (the default).

Each line is `<name> -- owner: <name> -- updated <timestamp>` (or `--
shared with <team name> --` when shared). Present these to the user as
a short summary, not the raw command output verbatim.

### Errors

- Any non-zero exit with an `Error: ...` line on stderr — surface that
  message directly. A `401`/session-related failure will say to run
  `rainbow login` again; don't attempt to fix this yourself, tell the
  user.
