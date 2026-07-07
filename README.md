# rainbow-skills

Agent skills for talking to [`rainbow-backend`](https://github.com/thomaskwan/rainbow-backend).
Install a skill into a project with:

```bash
npx skills add ptk-studio/rainbow-skills --skill <skill-name>
```

## Why this exists

Rainbow's product direction is a set of **workflows** — packaged,
app-like automations a user configures and runs to automate their own
business functions, each tied to one of their projects (see
`rainbow-frontend/features/pending/260706191522-feature-frontend-workflows-platform.md`
for the in-progress design). Once workflows exist, the point of this
repo is for an agent to be able to **drive them** directly — browse
what's available, configure one, kick off a run, check its status — the
same things a user would otherwise do by clicking around the Rainbow UI.

For now, before any workflow endpoints exist, this repo starts with a
single **read-only sample skill** listing existing projects, to prove
out the calling convention before there's anything to automate.

## Authentication

Skills here don't call `rainbow-backend` directly, and don't handle any
token themselves. Each one drives
[`rainbow-cli`](https://github.com/ptk-studio/rainbow-cli) — the same
pattern `vercel-labs/agent-skills`' deploy skill uses for the `vercel`
CLI, and `supabase/agent-skills` uses for the `supabase` CLI. `rainbow`
owns its own `login` (real Google OAuth, browser-based) and stores the
session itself; a skill just checks `rainbow whoami` and runs `rainbow
login` if needed, same as the reference skills check `vercel whoami` /
the Supabase MCP connection.

(An earlier version of this README described skills calling the API
directly with a manually-obtained token — superseded once `rainbow-cli`
existed to do this properly instead.)

## Skills

- **[list-projects](skills/list-projects/SKILL.md)** — list the
  signed-in user's Rainbow projects via `rainbow projects list`. Sample
  skill for now — see its `SKILL.md` for the exact flow.

## Skill structure

Each skill follows the [Agent Skills](https://agentskills.io/) format
(matching `supabase/agent-skills` and `vercel-labs/agent-skills`):

- `SKILL.md` — required skill manifest with frontmatter (`name`,
  `description`).
- `references/` — optional, for anything too long to keep inline in
  `SKILL.md` itself.
