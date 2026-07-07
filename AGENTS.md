# Agent guidelines — 88eggs-skills

A repo of `SKILL.md` files (the [Agent Skills](https://agentskills.io/)
format) — no application code. See `README.md` for why this repo exists
and how it relates to `88eggs-cli`/`88eggs-backend`/`88eggs-frontend`.

## Skills drive `88eggs-cli`, never call the API or handle a token directly

This is the one rule that matters most here, and the reason this repo
looks the way it does: a skill's job is to orchestrate the `88eggs` CLI
(`88eggs login`/`whoami`/`projects list`/etc.), the same way
`vercel-labs/agent-skills`' deploy skill drives the `vercel` CLI and
`supabase/agent-skills` drives the `supabase` CLI. Concretely:

- Never write a skill that calls `88eggs-backend`'s REST API with
  `curl`/`fetch` directly, or that asks the user to paste a token into
  an env var. If `88eggs-cli` doesn't yet have a command for something
  a new skill needs, that's a `88eggs-cli` feature to add first, not a
  reason for the skill to route around it.
- A skill checks auth state via `88eggs whoami` (fast, no network call
  per `88eggs-cli`'s own design) before doing real work, and only runs
  `88eggs login` after telling the user and getting their go-ahead —
  it opens their browser for a real Google sign-in, which needs their
  own interaction and isn't something to trigger silently.
- Never log, print, or echo anything from `~/.88eggs/credentials.json`.
  A skill has no reason to read that file directly — everything it
  needs comes back through `88eggs`'s own command output.

## `SKILL.md` format

- YAML frontmatter: `name` (matches the directory name) and
  `description` (specific enough that an agent can tell when to use it
  from the description alone — see the existing `list-projects` skill
  and the two reference repos linked from `README.md` for the bar).
- Body: numbered steps an agent follows, in order — not prose describing
  the feature. Include exact commands to run and how to interpret their
  output/exit codes, not just what the command does.
- `references/` subdirectory for anything long enough to clutter
  `SKILL.md` itself (a new skill probably doesn't need one yet — don't
  add the directory speculatively).

## Adding a new skill

1. `mkdir skills/<name>` and write `skills/<name>/SKILL.md`.
2. Add it to the `## Skills` list in `README.md` with a one-line
   description.
3. If it needs a `88eggs-cli` command that doesn't exist yet, add that
   to `88eggs-cli` first (see that repo's own `AGENTS.md`), verify it
   manually, then write the skill against the real, working command —
   don't write a skill against a command you're assuming will exist.

## Before committing

- No test suite here (there's no code to test) — the bar is: does the
  `SKILL.md` accurately describe commands that actually exist and
  actually behave the way it says? Run through the skill's own steps by
  hand at least once after writing or changing one.
