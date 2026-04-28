---
description: Run the full Prowlo workflow on a single Reddit opportunity — qualify, draft, and send to the user's dashboard for review
argument-hint: "<opportunity-id>"
---

# /prowlo-engage

Drive the end-to-end engagement workflow for one opportunity: pull the post, qualify the subreddit, read the thread, draft a reply tuned to the community's tolerance, and submit the draft to Prowlo for human review.

The argument is a Prowlo opportunity ID. If it is missing or not numeric, ask once and stop.

## Steps

1. **Pull the opportunity.** Call `get_opportunity` with the ID. Capture: title, body excerpt, opportunity score, risk score, subreddit name, post URL.

2. **Read the subreddit.** Call `get_subreddit_intelligence` for that subreddit. Surface promotion tolerance, removal rate, and engagement window in two short lines.

3. **Decide whether to continue.** If risk > 70 and opportunity < 70, recommend skipping and ask the user whether to proceed anyway. Wait for confirmation.

4. **Read the thread context.** Call `read_comments` for the opportunity. Skim the OP's replies and the top existing comments to pick up tone, what the OP is responding to, and whether any commenter has already given the answer the user would give. If they have, recommend not duplicating.

5. **Pull the user's voice.** Call `get_product_profile` if not already cached in this conversation. Use the voice and engagement guidelines verbatim — this is the user's positioning, not yours.

6. **Draft the reply.** Calibrate the draft to the subreddit's tolerance:
   - **Strict (low promotion tolerance, high removal rate):** lead with help. Mention the product only if it is the natural answer to the OP's question. Match the existing tone in the thread.
   - **Permissive:** be direct, concise, and useful. The product can be mentioned by name with a link if it adds value.
   - **Mixed:** lead with help; offer the product only when invited.
   Show the draft to the user inline before submitting.

7. **Submit the draft.** Once the user approves the draft, call `submit_draft` and confirm to the user that the reply has been sent to their Prowlo dashboard for review. Make the next action clear: "Open your Prowlo dashboard to review and post when you're ready."

## Guardrails

- Never call `submit_draft` without showing the user the final draft text first. Approval must be explicit.
- Never claim the reply has been posted to Reddit. The Prowlo MCP cannot post; it only stores drafts for human review.
- If any of `get_opportunity`, `get_subreddit_intelligence`, or `get_product_profile` errors, surface the error and stop. Do not draft a reply on partial data.
- Quote the actual score numbers, not adjectives, when explaining your recommendation.
