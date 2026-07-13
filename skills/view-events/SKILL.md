---
name: view-events
description: Browse the 88eggs activity log -- tasks starting/finishing, assets being added, apps installed/uninstalled, workers picking up tasks -- by driving the 88eggs CLI. Use when the user asks what happened, what's the recent activity, or wants to check the event/activity log for a project or across their account.
---

# View Events

Browses 88eggs' **event log** — a durable, append-only record of "a Task
started," "a Task finished," "an Asset
was added," "an App was installed," "an App was uninstalled," "a Worker
started a task," "a Worker finished a task" — by driving the
[`88eggs` CLI](https://github.com/ptk-studio/88eggs-cli), the same way
`vercel-labs`'s deploy skill drives the `vercel` CLI. This skill never
handles a token directly, and never calls `88eggs-backend` with
`curl`/`fetch` itself; `88eggs` owns its own login and every API call.

Events are produced automatically by the backend (database triggers on
Tasks/Assets/Apps/Pipelines) — nothing here creates or modifies an event, this
skill is read-only.

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

Once they confirm, run `88eggs login` and wait for it to complete
before continuing.

## Step 3 (optional): See what kinds of events exist

```bash
88eggs events types
```

Prints one line per known event type: `<key> -- <label> -- <description>
-- fields: <payload field names>`. Useful if the user asks "what can I
even filter by" — the `<key>` values (`task_started`, `task_finished`,
`asset_added`, `app_installed`,
`app_uninstalled`, `worker_started`, `worker_finished`) are exactly what
`--type` in Step 4 takes. Don't hardcode this list when reasoning about
what's available — run `events types` to see the live catalog, since new
types get added over time.

## Step 4: List events

```bash
88eggs events list [--project <projectId>] [--type <eventTypeKey>] [--page <page>] [--limit <limit>]
```

- Omit `--project` to see events across every project the caller can
  access; pass it to scope to one (run `88eggs projects list` first if
  the user named a project by name rather than id — see the
  `list-projects` skill).
- Omit `--type` for everything; pass one of the keys from Step 3 to
  filter to just that kind of event (e.g. the user asks "what images
  got generated recently" → `--type asset_added`).
- Defaults to page 1 at a reasonable page size; only pass `--page`/
  `--limit` if the user is paging through a long list.

Each line is `<eventId> -- <eventTypeKey> -- task <taskId> -- <createdAt>`
(the `task <taskId>` segment is omitted for an event with no associated
task). Events print newest-first.

## Step 5: Report the result

Summarize what happened in plain language rather than dumping the raw
command output — e.g. "3 tasks finished and 2 images were added in the
last few events" rather than reprinting every line verbatim, unless the
user specifically wants the raw list. If they ask about one specific
run or job's activity, cross-reference the `run_id`/`entity_id` against
`88eggs runs status <runId>` (see the `run-workflow` skill) for the
fuller picture rather than trying to explain everything from the event
log's own short summary line.

### Errors

- Any non-zero exit with an `Error: ...` line on stderr — surface that
  message directly.
- `No events found.` — a real, valid empty result, not an error; tell
  the user there's no matching activity rather than treating it as a
  failure.
- A `401`/session-related failure will say to run `88eggs login` again;
  don't attempt to fix this yourself, tell the user.
