---
description: Find recent mentions of a brand and its competitors and summarize sentiment. Use when you want to know what people say about your product or a competitor.
argument-hint: "<brand> [competitor ...] [--days N]"
---

# /prowlo-mentions

Find and summarize mentions of a brand and optional competitors, following the `brand-and-competitor-monitoring` skill.

Arguments: `$ARGUMENTS`

## Parse the arguments

- First name: the brand. Any further names: competitors. Quote multi-word names.
- `--days N`: the period. Default 30.
- No brand given: offer to use the product name and competitors from `get_product_profile`, or ask for a name, then stop until the user answers.

## Steps

1. `keyword_monitor_list` to see which names already have a daily Reddit-wide search.
2. For each name, `search_dataset` with `mode` "keyword", `query` set to the name, `any` set to the name and its obvious variants, and `from` set to N days ago.
3. If a name has little or no Dataset coverage, run one `social_search` (`platform` "reddit", `query` set to the name, `sort` "new", `time` "month") and, if relevant, one with `platform` "hackernews". Keep live reads to two per name.
4. Open the most important posts with `get_record` or `social_get_post` before judging sentiment.
5. Report: coverage per name, sentiment split with counts, main praise and complaints with links, comparison threads, and open questions.

## Writes

This command only reads. At the end, if a name has no Keyword Monitor, you may offer once to set up tracking: `keyword_monitor_create` for a daily search, and `alert_create` for Slack or webhook notifications (check `get_account` for integrations first). Show the exact settings and wait for a yes before each call.
