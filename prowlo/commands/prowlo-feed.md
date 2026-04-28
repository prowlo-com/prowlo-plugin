---
description: Show today's top-scored Reddit opportunities from Prowlo
argument-hint: "[r/<subreddit> | <search-text> | --opportunity HIGH|MEDIUM|LOW | --risk HIGH|MEDIUM|LOW | --relevance high|medium|all | --limit <n> | --page <n>]"
---

# /prowlo-feed

Pull the user's current Reddit opportunity feed and present it for prioritization. The user wants a fast, scannable view — not a wall of JSON.

## Steps

1. Parse the optional argument. Map flexibly to `get_opportunities` parameters:
   - A subreddit name like `r/SaaS` or `SaaS` → not a server-side filter; pull a wider page and filter the result by `subreddit` after.
   - A bare phrase → pass as `keywords` (server-side full-text search across post title and body).
   - `--opportunity HIGH|MEDIUM|LOW` → `opportunityThreshold` (defaults to user's profile preference).
   - `--risk HIGH|MEDIUM|LOW` → `riskThreshold` (defaults to user's profile preference; LOW means cap at low risk).
   - `--relevance high|medium|all` → `relevancePreset` (controls product-relevance filter).
   - `--limit <n>` → `limit` (1–50, default 10).
   - `--page <n>` → `page` (default 1).
   - No argument → call with no params, which uses the user's saved preferences.

2. Call `get_opportunities`. State the resolved parameters in one line so the user can adjust.

3. Render results as a compact table:

   | # | Opp | Risk | Subreddit | Title (truncated) | Intent | Reply type | postId (short) |

   - `Opp` and `Risk` are the enum levels (HIGH/MEDIUM/LOW) returned in `opportunity` and `risk`.
   - `Intent` is `signals.intentDirection` (BUYING / SELLING / DISCUSSING).
   - `Reply type` is `replyType` (PRODUCT_MENTION, EXPERIENCE_SHARE, ANSWER, etc.).
   - `postId (short)` is the first 8 chars of the UUID — full UUIDs are unwieldy. Always keep the full UUID available so the user can copy it for `/prowlo-engage`.
   - Truncate titles to 60 chars. Sort by `opportunity` (HIGH first), then by `risk` (LOW first).

4. End with one suggested next step. Pick the row with the best opp/risk ratio and surface it: *"Engage with `bf266d7c…` (HIGH opp, LOW risk, r/entrepreneur)? Run /prowlo-engage bf266d7c-a876-4ec9-b123-b65be7220cb3."*

## Guardrails

- If `opportunities` is empty, do not pull more pages. Suggest creating a tracked keyword (`keyword_create`) or broadening the product profile.
- If the trial has expired or scope is missing, surface the MCP error verbatim with the billing or settings link.
- Never call `get_opportunity`, `read_comments`, or `submit_draft` from this command — those belong to `/prowlo-engage`. This command is read-only on the feed.
- Do not invent UUIDs. Only show postIds the API returned.
