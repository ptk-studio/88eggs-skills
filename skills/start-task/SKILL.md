---
name: start-task
description: Browse the 88eggs task catalog, configure and start a task, and follow it through to completion by driving the 88eggs CLI. Use when the user asks to run, start, kick off, or configure an 88eggs task, or asks what tasks/automations are available.
---

# Start Task

Drives an 88eggs **task** (one packaged, app-like automation run against
one of the caller's projects) end to end — browse the task
catalog, gather parameters, start the task, poll it to completion, and
report the result — by driving the
[`88eggs` CLI](https://github.com/ptk-studio/88eggs-cli), the same way
`vercel-labs`'s deploy skill drives the `vercel` CLI. This skill never
handles a token directly, and never calls `88eggs-backend` with
`curl`/`fetch` itself; `88eggs` owns its own login and every API call.

## Step 1: Check the CLI is installed

```bash
command -v 88eggs
```

If missing, install it — it's a published npm package, a one-line global
install with no build step:

```bash
npm install -g 88eggs-cli
```

If this fails (no npm, permission error, etc.), tell the user rather
than working around it.

## Step 2: Check sign-in state

```bash
88eggs whoami
```

- Prints the signed-in email and exits `0` -> already signed in, go to
  Step 3.
- Prints `Not signed in. Run \`88eggs login\`.` and exits `1` -> not
  signed in.

If not signed in, **tell the user and ask before proceeding** — `88eggs
login` opens their browser for a real Google sign-in:

```
You're not signed in to 88eggs. I can run `88eggs login`, which will
open your browser to sign in with Google. Want me to proceed?
```

Once they confirm, run `88eggs login` and wait for it to complete
before continuing.

## Step 3: Find the task

```bash
88eggs tasks list
```

Each line is `<slug> -- <name> -- <description>`. If the user already
named one, match it against `<name>`/`<slug>`/`<description>`. If it's
ambiguous or they just asked "what can I run," show them the list and
ask which one.

If the catalog is empty, tell the user — there's nothing to start yet,
don't invent a task.

## Step 4: Look at its parameter spec

```bash
88eggs tasks show <slug>
```

Prints the definition's description and every parameter as `--param
<name>=<value>  (<type><, required>, default: <default>)`, with
`-- options: a, b` appended for `select` parameters. Every parameter has
a default, so **you don't need to ask the user about every field** —
use your own judgment for parameters with a sensible default (e.g.
leave `model=fast` alone unless they asked for higher quality), and only
ask the user about ones where their request clearly implies a specific
value (e.g. they described what they want the `prompt` to be).

## Step 5: Start the task

```bash
88eggs tasks start <slug> [--project <projectId>] [--name <name>] [--param key=value ...]
```

- Omit `--project` to use the caller's oldest project (the endpoint's
  own default); only pass it if the user named a specific project, and
  confirm which project first if it's ambiguous (run
  `88eggs projects list` to check).
- `--name` is a label for the task, shown in `task-runs list`/`task-runs status`
  and under each asset it produces — pass one if the user gave a
  name, or if a descriptive name would help them find it later among
  many; otherwise omit it and the backend generates one
  ("`<task name> <random word>`") rather than leaving it
  blank.
- Repeat `--param key=value` for every field you're overriding; leave
  the rest to fall back to their defaults (Step 4).

On success this prints `Task run <id> "<name>" queued (task:
<slug>, project: <projectId>).` — note the task id for the next step.

## Step 6: Poll until it finishes

```bash
88eggs task-runs status <taskRunId>
```

Prints `Status: queued|accepted|running|succeeded|failed`, plus, as the
worker fills them in: `Model: <model> (<provider>)`, `Cost: $<cost>`,
`Result asset: <assetId>`, `Error: ...` on failure, and an
`Outputs: {...}` line if the task produced outputs (the produced asset
id rides in there too; non-asset results like a published video URL as
well). Re-run this every few seconds — a couple of seconds apart is
reasonable, don't hammer it in a tight loop — until the status is
`succeeded` or `failed`.

## Step 7: Report the result

- **Succeeded**: tell the user it's done, then report what it produced —
  a task can produce an **asset**, non-asset **outputs**, or both:
  - If there's a `Result asset: <assetId>` line, mention you can pull it
    up with `88eggs assets show <assetId>` (see the `manage-assets`
    skill) if they want to see it. Some tasks record more than one
    produced asset in `Outputs:` (e.g. a song plus its captions) —
    report every asset id you see there rather than assuming exactly
    one.
  - If `Outputs: {...}` carries a non-asset result (e.g. a published
    video URL), surface it directly — don't assume every task ends in
    an asset.
- **Failed**: surface the `Error: ...` line. Don't guess at a fix or
  retry silently; ask the user how they'd like to proceed.

### Errors

- Any non-zero exit with an `Error: ...` line on stderr — surface that
  message directly.
- `88eggs tasks show`/`start` failing with "No task
  definition found with slug ..." means the slug doesn't match the
  catalog — re-check Step 3, don't guess at a close spelling.
- A `401`/session-related failure will say to run `88eggs login` again;
  don't attempt to fix this yourself, tell the user.
