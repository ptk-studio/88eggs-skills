---
name: manage-schedules
description: Look at Pipeline Sets and their runs, and create, change or remove recurring schedules for a pipeline or an active Pipeline Set, by driving the 88eggs CLI. Use when the user asks what sets or runs exist, how a run is going or what it cost, or wants something to run on a schedule (daily, weekly, a cron).
---

# Manage Schedules, Sets and Runs

Reads **Pipeline Sets** (a chain of pipelines with a cost ceiling) and their
**runs**, and manages **schedules** (a cron that starts a pipeline or an
active set on a cadence) by driving the
[`88eggs` CLI](https://github.com/ptk-studio/88eggs-cli). It never calls the
API directly and never handles a token.

**What this skill does not do:** start a paid run, publish or unpublish a set,
edit a set, or touch offers, orders or credits. A schedule *does* cause runs to
start later, and runs cost money, so creating or enabling one needs the user's
clear yes (Step 5).

## Step 1: Check the CLI and its version

```bash
command -v 88eggs && 88eggs pipeline-sets --help >/dev/null 2>&1 && echo ok
```

- No `88eggs`: `npm install -g 88eggs-cli`. If that fails, tell the user; don't
  sudo or build from source.
- `88eggs` exists but `pipeline-sets` is unknown: the installed CLI is older
  than the set commands. Tell the user to update (`npm install -g 88eggs-cli`)
  and stop. Don't fall back to `curl`.

## Step 2: Check sign-in

```bash
88eggs whoami
```

Exit `0`: continue. "Not signed in": **tell the user to run `88eggs login`
themselves** (it asks for their password) and wait; never run it yourself.

## Step 3: Look at sets (read-only)

```bash
88eggs pipeline-sets list [--project <projectId>] [--status active] [--query <text>]
88eggs pipeline-sets show <idOrSlug> [--project <projectId>]
88eggs pipeline-sets validate <idOrSlug>      # exit 1 when it does not validate
```

- `show` prints the steps, inputs, the **cost ceiling per run**, an estimate
  ("about" when every step has history, "at least" when some do not) and
  whether it validates. Quote the ceiling and estimate when talking about cost.
- A slug can exist in two projects: if the CLI says so, ask which and pass
  `--project`.
- Only an `active` set can be scheduled. A `draft` or `archived` one cannot;
  the user activates a set in the app.

## Step 4: Look at runs (read-only)

```bash
88eggs pipeline-set-runs list [--set <idOrSlug>] [--status paused|failed|running|completed] [--needs-me]
88eggs pipeline-set-runs status <runId>       # cost against the ceiling, each step
88eggs pipeline-runs list [--project <projectId>]      # single-pipeline runs
88eggs pipeline-runs status <pipelineRunId>
```

Report status, cost against the ceiling (`$0.13 of $2.00`), and the failed or paused
step and its message **word for word**. `--needs-me` lists runs waiting on the
user. Don't approve, retry or re-run a set run for them; say what is waiting.

## Step 5: Schedules

Read first:

```bash
88eggs schedules list [--project <projectId>]
88eggs schedules show <scheduleId>
```

**Create** (writes; do it only after the user has said yes to this exact
schedule). Before asking, say in one message: what runs (pipeline or set, its
ceiling and estimate from Step 3), the cron in plain words **and the
timezone**, the project, and that it will keep starting runs until disabled.

```bash
# a pipeline:
88eggs schedules create --definition <slug> --cron "0 9 * * *" --name "<name>" \
    [--timezone America/New_York] [--project <projectId>] [--param key=value ...] [--disabled]
# an ACTIVE Pipeline Set (exactly one of --definition and --set):
88eggs schedules create --set <slug> --cron "0 9 * * *" --name "<name>" [same options]
```

- `--disabled` creates it paused; prefer it when the user wants to review first.
- Give every required input of the set through `--param`. Don't guess values.
- A scheduled set run starts without pausing between steps and is stopped by the
  daily cost ceiling. Say so.
- If the API refuses (set not active, bad cron, unknown slug), show its message
  and stop; don't try another route.

**Change or remove** (also after the user's yes):

```bash
88eggs schedules update <scheduleId> [--name] [--cron] [--timezone] [--enable|--disable] [--param k=v ...]
88eggs schedules delete <scheduleId>
```

Prefer `--disable` over `delete` when the user only wants it to stop. Delete only
when they ask, naming the schedule.

## Step 6: Report

Say what you ran and what came back: ids, names, status, cost, next steps for the
user. Never print anything from `~/.88eggs/credentials.json`, a token, or a
signed asset URL.
