---
description: Show today's top-scored Reddit opportunities from Prowlo
argument-hint: "[subreddit | keyword | --risk <n> | --limit <n>]"
---

# /prowlo-feed

Pull the user's current Reddit opportunity feed and present it for prioritization. The user wants a fast, scannable view — not a wall of JSON.

## Steps

1. Parse the optional argument. Accept these forms:
   - A subreddit name like `r/SaaS` or `SaaS` → filter by that subreddit.
   - A bare keyword string → filter by keyword match.
   - `--risk <n>` → cap risk score at `n` (default 60).
   - `--limit <n>` → return at most `n` opportunities (default 10).
   - No argument → use defaults: last 24 hours, risk ≤ 60, limit 10.

2. Call `get_opportunities` with the resolved filters. State the filter set you used in one line so the user can adjust.

3. Render the results as a compact table:

   | # | Score | Risk | Subreddit | Title (truncated) | ID |

   Sort by opportunity score descending. Truncate titles to 60 chars.

4. End with a single suggested next step in plain language, e.g. *"Reply to ID 8421 (score 87, r/SaaS)? Run /prowlo-engage 8421."*

## Guardrails

- If the feed is empty, do not pull more data. Suggest creating a tracked keyword via `keyword_create`, or broadening the product profile.
- If the trial has expired or scope is missing, surface the MCP error verbatim with the billing or settings link.
- Never call `get_opportunity` or `submit_draft` from this command — those belong to `/prowlo-engage`. This command is read-only on the feed.
