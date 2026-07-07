# 88eggs-skills

Agent skills for talking to `88eggs-backend` (private repo, not linked
here). Install a skill into a project with:

```bash
npx skills add ptk-studio/88eggs-skills --skill <skill-name>
```

## Why this exists

88eggs' product direction is a set of **workflows** — packaged,
app-like automations a user configures and runs to automate their own
business functions, each tied to one of their projects (see
`88eggs-frontend/features/pending/260706191522-feature-frontend-workflows-platform.md`
for the in-progress design). Once workflows exist, the point of this
repo is for an agent to be able to **drive them** directly — browse
what's available, configure one, kick off a run, check its status — the
same things a user would otherwise do by clicking around the 88eggs UI.

For now, before any workflow endpoints exist, this repo starts with a
single **read-only sample skill** listing existing projects, to prove
out the calling convention before there's anything to automate.

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
  signed-in user's 88eggs projects via `88eggs projects list`. Sample
  skill for now — see its `SKILL.md` for the exact flow.

## Skill structure

Each skill follows the [Agent Skills](https://agentskills.io/) format
(matching `supabase/agent-skills` and `vercel-labs/agent-skills`):

- `SKILL.md` — required skill manifest with frontmatter (`name`,
  `description`).
- `references/` — optional, for anything too long to keep inline in
  `SKILL.md` itself.
