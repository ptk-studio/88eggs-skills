---
name: list-projects
description: List the signed-in user's Rainbow projects (owned and team-shared) by calling rainbow-backend's GET /projects. Use when the user asks to see, list, or check their Rainbow projects.
---

# List Projects

Fetches the caller's own Rainbow projects — owned and shared with any
team they belong to — from `rainbow-backend`.

## Prerequisites

Two environment variables:

- `RAINBOW_API_URL` — the backend's base URL (e.g.
  `https://rainbow-backend-two.vercel.app`, or `http://localhost:3000`
  for local development).
- `RAINBOW_API_TOKEN` — a bearer token for the signed-in user.

If either is missing, tell the user what's needed rather than guessing —
don't invent a URL or attempt the request without a token.

**On `RAINBOW_API_TOKEN` today**: rainbow-backend doesn't yet have a
long-lived credential a skill can use on its own (no OAuth/API-key
flow — see `rainbow-backend/features/pending/` for the proposed Personal
Access Token feature). Until that ships, this needs a real Supabase
session access token, which expires in about an hour — fine for trying
this skill, not for anything unattended. If the user doesn't have one
handy, tell them to sign in to the Rainbow frontend and copy their
session's access token, rather than trying to obtain one yourself.

## Fetching projects

```bash
curl -s "$RAINBOW_API_URL/projects" \
  -H "Authorization: Bearer $RAINBOW_API_TOKEN"
```

Optionally scope the list with a query param — `?scope=mine` (owned
only) or `?scope=shared` (shared with a team only); omit for everything
the caller can see (the default).

### Response shape

```json
{
  "projects": [
    {
      "id": "string",
      "name": "string",
      "description": "string | null",
      "created_at": "ISO 8601 timestamp",
      "updated_at": "ISO 8601 timestamp",
      "owner_id": "string",
      "team_id": "string | null",
      "owner": { "id": "string", "full_name": "string | null" },
      "team": { "id": "string", "name": "string" } | null
    }
  ]
}
```

### Errors

- `401 { "error": "Unauthorized" }` — missing/invalid/expired token. Tell
  the user their token likely expired (Supabase session tokens are
  short-lived) and to get a fresh one, rather than retrying the same
  request.
- Any other non-2xx: surface the response body's `error` message
  directly rather than a generic failure.

## Presenting results

Summarize as a short list (name, owner or team if shared, last updated)
rather than dumping the raw JSON — this mirrors how `/projects` renders
in the Rainbow frontend itself (name, ownership badge, team, relative
"updated" time).
