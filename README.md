# 88eggs-skills

Agent skills for talking to `88eggs-backend` (private repo, not linked
here). Install a skill into a project with:

```bash
npx skills add ptk-studio/88eggs-skills --skill <skill-name>
```

## Why this exists

88eggs' product direction is a set of **task definitions** — packaged,
app-like automations a user configures and starts to automate their own
business functions, each task tied to one of their projects (this began
as "workflows"; see
`88eggs-frontend/features/pending/260706191522-feature-frontend-workflows-platform.md`
for the original design). The point of this repo is for an agent to be
able to **drive them** directly — browse the catalog, configure one,
start a task, check its status — the same things a user would otherwise
do by clicking around the 88eggs UI.

This started with a single **read-only sample skill** listing existing
projects, to prove out the calling convention before there was anything
to automate. Now that `88eggs-backend` has shipped the assets API and the
tasks framework (a task-definition catalog + tasks + queue/worker —
originally the "workflows framework", see
`88eggs-backend/features/completed/260707075756-feature-backend-workflows-framework.md`;
`88eggs-backend/docs/TASK_DEFINITIONS.md` is the live catalog doc),
this repo also has skills to browse/organize assets and to actually run
a task end to end. Image Generator Task was the first concrete
catalog entry (image generation via Gemini or OpenAI's `gpt-image-1`),
so `start-task` drives a real catalog entry, not an empty one. `88eggs-backend` also shipped Events (see
`88eggs-backend/features/completed/260707161500-feature-backend-events.md`)
— a durable log of that same task/asset activity — so `view-events`
can answer "what happened" without re-deriving it from `tasks status`
calls one at a time.

## Authentication

Skills here don't call `88eggs-backend` directly, and don't handle any
token themselves. Each one drives
[`88eggs-cli`](https://github.com/ptk-studio/88eggs-cli) — the same
pattern `vercel-labs/agent-skills`' deploy skill uses for the `vercel`
CLI, and `supabase/agent-skills` uses for the `supabase` CLI. `88eggs`
owns its own `login` (real Google OAuth, browser-based) and stores the
session itself; a skill just checks `88eggs whoami` and runs `88eggs
login` if needed, same as the reference skills check `vercel whoami` /
the Supabase MCP connection.

`88eggs-cli` is published on npm (`npm install -g 88eggs-cli`), so a
skill can install it automatically when missing — same as
`deploy-to-vercel` installing a missing `vercel` CLI — rather than just
detecting it's absent and stopping.

(An earlier version of this README described skills calling the API
directly with a manually-obtained token — superseded once `88eggs-cli`
existed to do this properly instead.)

## Skills

- **[list-projects](skills/list-projects/SKILL.md)** — list the
  signed-in user's 88eggs projects via `88eggs projects list`.
- **[manage-assets](skills/manage-assets/SKILL.md)** — browse and
  organize 88eggs assets: list, view, tag, like, and move between
  projects.
- **[start-task](skills/start-task/SKILL.md)** — browse the
  task-definition catalog, configure and start a task, poll it to
  completion, and report the result.
- **[manage-apps](skills/manage-apps/SKILL.md)** — browse the app
  catalog (App Store) and manage a project's installed apps: install,
  list, show, uninstall, and list an app's pages.
- **[manage-data-tables](skills/manage-data-tables/SKILL.md)** — create
  and manage a project's data tables and their rows: create with
  columns, edit the schema, list/add/update/delete rows, archive/restore,
  and publish a table's schema as a template.
- **[view-events](skills/view-events/SKILL.md)** — browse the activity
  log (tasks starting/finishing, assets added, apps installed/
  uninstalled, workers picking up tasks), across every accessible
  project or scoped to one.

## Skill structure

Each skill follows the [Agent Skills](https://agentskills.io/) format
(matching `supabase/agent-skills` and `vercel-labs/agent-skills`):

- `SKILL.md` — required skill manifest with frontmatter (`name`,
  `description`).
- `references/` — optional, for anything too long to keep inline in
  `SKILL.md` itself.
