---
name: reddit-engagement
description: Use when the user asks anything about engaging on Reddit through Prowlo — finding leads on Reddit, drafting Reddit replies, qualifying a subreddit, deciding whether a thread is safe to engage with, managing tracked keywords, or handling Reddit lead generation workflows. Triggers on phrases like "find Reddit leads", "draft a Reddit reply", "is this subreddit safe", "what's my opportunity feed", "should I post in r/X", "Prowlo opportunities", "Reddit lead generation", "track a keyword".
---

# Reddit Engagement with Prowlo

This skill activates whenever the user is using Prowlo through Claude to engage on Reddit. It encodes the framework Prowlo uses to keep accounts and subreddits intact, and the workflow that turns an opportunity feed into a posted reply.

## Mental model

Reddit is not a broadcast channel. Every subreddit is its own community with unwritten rules, a tolerance threshold for self-promotion, and moderators who remove posts that read as marketing. A reply that lands well in r/Entrepreneur will be removed in r/SaaS. A reply that lands well in r/SaaS will get downvoted in r/startups. The Prowlo MCP gives Claude the data to tell those situations apart; the user gives final approval before anything is posted.

Three principles apply to every Prowlo workflow:

1. **Score before drafting.** Every opportunity has two independent scores: opportunity (intent) and risk (subreddit moderation tolerance). Drafting a reply for a low-opportunity or high-risk thread wastes time and accounts. Read both scores first.
2. **Read the subreddit before the post.** `get_subreddit_intelligence` returns the moderation pattern for the community — promotion tolerance, removal rate, mod strictness. This determines tone, whether the reply can mention Prowlo (or any product) at all, and whether engagement is worth attempting.
3. **Never auto-post.** `submit_draft` sends the reply to the user's Prowlo dashboard for human review. Do not promise to post for the user. Reddit posts are permanent and a wrong one can cost the account.

## The available tools

The Prowlo MCP server exposes 10 tools, grouped into two areas.

**Opportunity intelligence:**

- `get_opportunities` — list scored opportunities from the user's feed. Filters by subreddit, score range, recency, keyword. Use this for "show me opportunities", "today's leads", "high-intent threads in r/X".
- `get_opportunity` — full detail for a single opportunity, including the post body, scores, subreddit, and engagement guidance.
- `read_comments` — comments under a post. Use this to understand thread context before drafting a reply (existing reply tone, what the OP responded to, whether the conversation is still active).
- `get_subreddit_intelligence` — moderation pattern, promotion tolerance, removal rate, and engagement window for a subreddit. Use this whenever the user names a subreddit or before drafting any reply.
- `get_product_profile` — the user's product positioning, target audience, voice, and engagement guidelines. Read this before writing any reply so the draft matches the user's voice.
- `submit_draft` — send a reply draft to the user's Prowlo dashboard. The draft is reviewed and posted manually by the user. Never claim this posts to Reddit directly.

**Keyword management:**

- `keyword_list` — currently tracked keywords with match counts and last-match timestamps.
- `keyword_create` — add a keyword. Plan limits apply (Starter: 10, Growth: 30). New keywords backfill 7 days of matches automatically.
- `keyword_update` — activate or pause a keyword. Pausing preserves match history.
- `keyword_delete` — remove a keyword and its match history. Confirm with the user before deleting.

## The standard workflow: opportunity to draft

When the user wants to engage with a specific opportunity, run this sequence:

1. **Pull the opportunity:** `get_opportunity` with the ID. Read the post, both scores, and the subreddit name.
2. **Pull the subreddit:** `get_subreddit_intelligence` for that subreddit. Surface the promotion tolerance and removal rate to the user. If the subreddit is high-risk and the opportunity is mid-tier, recommend skipping rather than drafting.
3. **Read the thread context:** `read_comments` for that opportunity. Look at the OP's replies, the existing top comments, and the tone. A draft that ignores the conversation reads as marketing.
4. **Pull the user's voice:** `get_product_profile` if you don't already have it cached in conversation. The reply must sound like the user, not generic AI prose.
5. **Draft the reply** — explicitly aligned to the subreddit's tolerance level, the thread's tone, and the user's product profile. If the subreddit is strict, the reply leads with help and only mentions the product if it's genuinely the answer. If the subreddit is permissive, the reply can be more direct.
6. **Submit the draft:** `submit_draft`. Tell the user it's in their Prowlo dashboard for review and never claim it has been posted.

## Defaults and tone

When the user says "show me opportunities" without filters, default to: last 24 hours, sorted by opportunity score descending, risk under 60. State the filter you used so the user can adjust.

When you summarize a feed, surface the score numbers, not vague language. "Opportunity 8421 — score 87, risk 42, r/SaaS" is more useful than "a strong lead in a SaaS community." Numbers let the user trust your prioritization.

When risk is over 70, lead with the warning before discussing the post. When opportunity is under 50, ask whether to filter it out before pulling more data.

## What not to do

- Do not invent tool names. The 10 tools above are the only ones available.
- Do not treat `submit_draft` as posting to Reddit. It writes to Prowlo's database for human review.
- Do not produce drafts without reading the subreddit intelligence first.
- Do not mention the user's product in a draft for a strict subreddit unless the product genuinely solves the OP's problem and the moderation pattern allows it.
- Do not bulk-delete keywords or submit multiple drafts in a single turn without explicit confirmation per item.

## Edge cases

- **No active feed:** if `get_opportunities` returns empty, suggest creating a tracked keyword (`keyword_create`) or adjusting the product profile to broaden semantic matching, rather than pulling more data.
- **Trial expired:** the MCP returns a clear error. Surface it to the user with the link to billing. Do not retry.
- **Rate or scope errors:** the MCP returns scope/rate errors with explanations. Surface them verbatim and ask the user how to proceed.
