---
description: Run the full Prowlo workflow on a single Reddit opportunity — qualify, draft, and send to the user's dashboard for review
argument-hint: "<postId-uuid>"
---

# /prowlo-engage

Drive the end-to-end engagement workflow for one opportunity: pull the post, qualify the subreddit, read the thread, draft a reply tuned to the community's tolerance, and submit the draft to Prowlo for human review.

The argument is a Prowlo opportunity `postId` — a UUID like `bf266d7c-a876-4ec9-b123-b65be7220cb3`. If it is missing or not a valid UUID, ask once and stop. Do not attempt with a guessed or numeric ID.

## Steps

1. **Pull the opportunity.** Call `get_opportunity({ postId })`. Capture: title, body excerpt, `opportunity` (HIGH/MEDIUM/LOW), `risk` (HIGH/MEDIUM/LOW), `subreddit`, `url`, `signals` (intentDirection, freshness, saturation), `guidance` (tips, warnings, recommendedApproach, engagementWindow, confidence), `replyType`, `replyTypeHint`, `autoDraft` (if present, Prowlo has already pre-drafted something — start from that, don't redraft from scratch).

2. **Read the subreddit.** Call `get_subreddit_intelligence({ subreddit })`. Don't trust the `strictnessLevel` label alone — many "lenient" subreddits enforce self-promotion strictly. Surface to the user, in two short lines:
   - `survivalRates.selfMention` (the percent of self-mention posts that survive removal)
   - `accountReadiness.inferredMinAccountAgeDays` and `inferredMinKarma`
   - Top 3 `triggerWords` by `riskScore` if any are likely to appear in the draft.

3. **Decide whether to continue.** Hard-skip rules — if any apply, recommend skipping and ask the user whether to proceed anyway. Wait for confirmation:
   - `risk` is HIGH and `opportunity` is anything below HIGH.
   - `signals.intentDirection` is SELLING (the OP is promoting their own thing — engagement rarely lands).
   - `survivalRates.selfMention` is below 0.20 AND `replyType` is PRODUCT_MENTION (the subreddit will remove the kind of reply you would draft).

4. **Read the thread context.** Call `read_comments({ postId })`. Skim the OP's replies and the top existing comments to catch tone, what the OP is responding to, and whether someone has already given the answer the user would give. If they have, recommend not duplicating.

5. **Pull the user's voice.** Call `get_product_profile` if not already cached in this conversation. Use the voice and engagement guidelines verbatim — this is the user's positioning, not yours.

6. **Draft the reply.** Calibrate to the subreddit's `survivalRates.selfMention` and the `replyType` hint:
   - **selfMention < 0.20 (strict on promotion):** lead with help. Mention the product only if it is the natural answer to the OP's question. Avoid the high-`riskScore` trigger words from the subreddit profile. Match the existing tone in the thread.
   - **selfMention 0.20–0.50 (mixed):** lead with help; offer the product only when invited or when `replyType` is EXPERIENCE_SHARE.
   - **selfMention > 0.50 (permissive):** be direct, concise, and useful. Follow `replyTypeHint` literally.
   Show the draft to the user inline before submitting. Quote the actual numbers ("self-mention survival 6.6%") when explaining your tone choice.

7. **Submit the draft.** Once the user approves, call `submit_draft` and confirm the reply has been sent to their Prowlo dashboard for review. Make the next action clear: "Open your Prowlo dashboard to review and post when you're ready."

## Guardrails

- Never call `submit_draft` without showing the user the final draft text first. Approval must be explicit.
- Never claim the reply has been posted to Reddit. The Prowlo MCP cannot post; it only stores drafts for human review.
- If any of `get_opportunity`, `get_subreddit_intelligence`, or `get_product_profile` errors, surface the error and stop. Do not draft a reply on partial data.
- Quote actual enum levels and percentages, not vague adjectives, when explaining your recommendation. "HIGH risk, self-mention survival 6.6%" beats "this subreddit is risky."
