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
single **read-only sample skill** against the API that already exists
(`GET /projects`), to prove out the calling convention before there's
anything to automate.

## Authentication

Every skill here calls `rainbow-backend` as a specific signed-in user, so
each needs a bearer token in the `Authorization` header — see each
skill's own `SKILL.md` for exactly which environment variable it reads.

**Today**, the only way to get a valid token is a real Supabase session
access token (e.g. copied out of a signed-in browser session) — it
expires in about an hour, which is fine for trying a skill out but not a
real workflow. A proper long-lived, revocable credential (a Rainbow
Personal Access Token) is proposed in
`rainbow-backend`'s `features/pending/` — skills here will move to that
once it ships.

## Skills

- **[list-projects](skills/list-projects/SKILL.md)** — list the
  signed-in user's Rainbow projects (calls `GET /projects`). Sample skill
  for now — see its `SKILL.md` for the exact request/response shape.

## Skill structure

Each skill follows the [Agent Skills](https://agentskills.io/) format
(matching `supabase/agent-skills` and `vercel-labs/agent-skills`):

- `SKILL.md` — required skill manifest with frontmatter (`name`,
  `description`).
- `references/` — optional, for anything too long to keep inline in
  `SKILL.md` itself.
