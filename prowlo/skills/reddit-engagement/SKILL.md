---
name: reddit-engagement
description: Use when the user asks anything about engaging on Reddit through Prowlo — finding leads on Reddit, drafting Reddit replies, qualifying a subreddit, deciding whether a thread is safe to engage with, managing tracked keywords, or handling Reddit lead generation workflows. Triggers on phrases like "find Reddit leads", "draft a Reddit reply", "is this subreddit safe", "what's my opportunity feed", "should I post in r/X", "Prowlo opportunities", "Reddit lead generation", "track a keyword".
---

# Reddit Engagement with Prowlo

This skill activates whenever the user is using Prowlo through Claude to engage on Reddit. It encodes the framework Prowlo uses to keep accounts and subreddits intact, and the workflow that turns an opportunity feed into a posted reply.

## Mental model

Reddit is not a broadcast channel. Every subreddit is its own community with unwritten rules, a tolerance threshold for self-promotion, and moderators who remove posts that read as marketing. A reply that lands well in r/Entrepreneur will be removed in r/SaaS. A reply that lands well in r/SaaS will get downvoted in r/startups. The Prowlo MCP gives Claude the data to tell those situations apart; the user gives final approval before anything is posted.

Three principles apply to every Prowlo workflow:

1. **Score before drafting.** Every opportunity carries `opportunity` (intent) and `risk` (subreddit moderation tolerance) levels — both `HIGH | MEDIUM | LOW` enums. Drafting for a low-opportunity or high-risk thread wastes time and accounts. Read both before doing anything.
2. **Trust the survival rate over the strictness label.** `get_subreddit_intelligence` returns `strictnessLevel` ("lenient", "moderate", "strict") AND `survivalRates.selfMention` (the percent of self-promo posts that survive removal). The label can lie; the rate cannot. r/startups is "lenient" overall but survives self-promotion at 6.6% — that is a strict subreddit for promotional replies.
3. **Never auto-post.** `submit_draft` sends the reply to the user's Prowlo dashboard for human review. Do not promise to post for the user. Reddit posts are permanent and a wrong one can cost the account.

## The available tools

The Prowlo MCP server exposes 10 tools, grouped into two areas.

**Opportunity intelligence:**

- `get_opportunities({ opportunityThreshold?, riskThreshold?, relevancePreset?, keywords?, limit?, page? })` — list scored opportunities. Thresholds are enums (HIGH/MEDIUM/LOW); `relevancePreset` is `high|medium|all`; `keywords` is a server-side full-text search over title + body. Returns `pagination` and an `opportunities` array.
- `get_opportunity({ postId })` — full detail for a single opportunity. `postId` is a UUID, not a number.
- `read_comments({ postId })` — comments under a post. Use this to understand thread context before drafting (existing reply tone, what the OP responded to, whether the conversation is still active).
- `get_subreddit_intelligence({ subreddit? })` — moderation pattern, rules, survival rates, trigger words, account readiness. Omit `subreddit` for an overview of all profiled subreddits.
- `get_product_profile` — the user's product positioning, target audience, voice, and engagement guidelines. Read this before writing any reply so the draft matches the user's voice.
- `submit_draft` — send a reply draft to the user's Prowlo dashboard. The draft is reviewed and posted manually by the user. Never claim this posts to Reddit directly.

**Keyword management:**

- `keyword_list` — currently tracked keywords with match counts and last-match timestamps.
- `keyword_create` — add a keyword. Plan limits apply (Starter: 10, Growth: 30). Note the API note: changes take effect on the next scheduled processing cycle, and existing opportunity scores are not retroactively updated.
- `keyword_update` — activate or pause a keyword. Pausing preserves match history.
- `keyword_delete` — remove a keyword and its match history. Confirm with the user before deleting.

## What the API actually returns (and how to use it)

`get_opportunities` and `get_opportunity` return rich fields. Treat these as the working set for any decision:

- `opportunity` and `risk` — `HIGH | MEDIUM | LOW`. The two scores that matter.
- `signals.intentDirection` — `BUYING | SELLING | DISCUSSING`. Skip SELLING posts (the OP is promoting their own thing; engagement rarely lands).
- `signals.intentStrength`, `signals.freshness`, `signals.saturation`, `signals.semanticRelevance` — context modifiers. Surface them when explaining a recommendation.
- `guidance.tips`, `guidance.warnings` — pre-computed actionable strings. Quote them; don't paraphrase them away.
- `guidance.recommendedApproach` — Prowlo's per-post engagement strategy. Treat as the default; deviate only with reason.
- `guidance.engagementWindow` — how long the thread is likely to stay active. Communicate it.
- `guidance.confidence` — `LOW | MEDIUM | HIGH` confidence in the guidance itself. If LOW, lower your own confidence in the recommendation.
- `replyType` — `PRODUCT_MENTION | EXPERIENCE_SHARE | ANSWER | etc.` Classifier for what kind of reply fits.
- `replyTypeHint` — one-line drafting instruction tied to `replyType`. Follow it literally.
- `autoDraft` — if non-null, Prowlo has already pre-drafted the reply. Start from that draft and refine; don't re-draft from scratch.

`get_subreddit_intelligence` returns:

- `strictnessLevel` — coarse label (`lenient | moderate | strict`). Useful for orientation, but never the deciding signal alone.
- `survivalRates.selfMention` — the actual percent of self-promotional posts that survive. **This is the deciding signal for "is it safe to mention the product."**
- `survivalRates.links` — same for posts containing links.
- `accountReadiness.inferredMinAccountAgeDays` and `inferredMinKarma` — age/karma thresholds the subreddit appears to enforce.
- `triggerWords` — phrases with their `riskScore`. Avoid the high-scoring ones in the draft.
- `rules` and `correlations` — full rules with `enforcement` levels and supporting evidence.

## The standard workflow: opportunity to draft

When the user wants to engage with a specific opportunity, run this sequence:

1. **Pull the opportunity:** `get_opportunity({ postId })`. Read the fields above.
2. **Pull the subreddit:** `get_subreddit_intelligence({ subreddit })`. Surface `survivalRates.selfMention` and `accountReadiness` thresholds. If selfMention is below 20% AND `replyType` is PRODUCT_MENTION, recommend skipping rather than drafting.
3. **Read the thread:** `read_comments({ postId })`. Check the OP's replies, the top existing comments, and the tone. A draft that ignores the conversation reads as marketing.
4. **Pull the user's voice:** `get_product_profile` if you don't already have it cached in conversation. The reply must sound like the user, not generic AI prose.
5. **Draft the reply** — calibrated to `survivalRates.selfMention`, the thread's tone, the `replyTypeHint`, and the user's product profile. If `autoDraft` exists, refine it instead of starting fresh.
6. **Submit the draft:** `submit_draft`. Tell the user it's in their Prowlo dashboard for review; never claim it has been posted.

## Defaults and tone

When the user says "show me opportunities" without filters, call `get_opportunities()` with no arguments — that uses the user's saved profile preferences. State the resolved filter set in one line so the user can adjust.

When you summarize a feed, surface the actual fields, not vague language. *"`bf266d7c…` — opp HIGH, risk MEDIUM, BUYING intent, r/entrepreneur, replyType PRODUCT_MENTION"* is more useful than *"a strong lead in an entrepreneur community."* The `postId` UUID is what the user pastes into `/prowlo-engage` — keep it visible.

When `risk` is HIGH, lead with the warning before discussing the post. When `opportunity` is LOW, ask whether to filter it out before pulling more data.

## What not to do

- Do not invent tool names. The 10 tools above are the only ones available.
- Do not invent or shorten `postId`s. They are UUIDs returned by the API. Show abbreviated forms for readability, but always pass the full UUID to `get_opportunity`, `read_comments`, and `submit_draft`.
- Do not invent numeric "scores" like 87 or 42. The API uses HIGH/MEDIUM/LOW enums, plus actual percentages for survival rates and risk scores. Quote the real values.
- Do not treat `submit_draft` as posting to Reddit. It writes to Prowlo's database for human review.
- Do not produce drafts without reading the subreddit intelligence first.
- Do not rely on `strictnessLevel` alone — always read `survivalRates.selfMention` before deciding the tone.
- Do not bulk-delete keywords or submit multiple drafts in a single turn without explicit confirmation per item.

## Edge cases

- **No opportunities returned:** if `get_opportunities` returns an empty `opportunities` array, suggest creating a tracked keyword (`keyword_create`) or relaxing `relevancePreset` to `medium` / `all`. Do not pull more pages.
- **Empty keyword list:** `keyword_list` returning `[]` is normal for new accounts. Suggest the user add their top product term, top competitor name, and one category phrase.
- **Trial expired:** the MCP returns a clear error directing to billing. Surface it to the user with the link. Do not retry.
- **Rate or scope errors:** the MCP returns scope/rate errors with explanations. Surface them verbatim and ask the user how to proceed.
