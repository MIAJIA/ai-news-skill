---
name: news
description: 个性化 AI/科技/产品新闻简报。从 RSS、Twitter/X、WebSearch 三条策略并行采集，AI 筛选 3-5 条最相关资讯，终端输出。可独立使用或被 /today 调用。
---

# /news

个性化新闻简报。并行采集 → AI 筛选 → 终端输出 3-5 条最相关资讯。

## Step 1: Fetch sources

Read source configuration from `references/news-sources.md` (relative to this skill's base directory).

Run **all three strategies in parallel**.

### Strategy A — RSS Feeds (primary, low-token)

Read all RSS URLs from `references/news-sources.md` (the "RSS Feeds" tables).

Use `WebFetch` to fetch all feeds **in parallel**, batched up to 5 concurrent calls:

- Batch 1: HN, Karpathy, DeepMind, Google Research, Ethan Mollick
- Batch 2: LangChain, Stratechery, Lenny, Paul Graham, Astral Codex Ten
- Batch 3: Joel on Software, Sebastian Raschka, fast.ai, Distill.pub, arXiv cs.AI
- Batch 4: Sam Altman, Dwarkesh Patel, Amjad Masad

For each feed:

- Extract **titles and dates only** from the last 48 hours
- Do NOT fetch full article content (saves tokens)
- If a feed fails, skip silently and continue

### Strategy B — Twitter/X via WebSearch (high-signal accounts)

Twitter has no public RSS. Use `WebSearch` to catch viral/high-engagement tweets from key accounts listed in `references/news-sources.md` (the "Twitter/X Accounts" tables).

Run **two** parallel `WebSearch` calls:

1. Scientists + Engineers:

```
query: "site:x.com (@_akhaliq OR @DrJimFan OR @polynoamial OR @karpathy_out OR @yoheinakajima) today"
```

2. Builders + Visionaries:

```
query: "site:x.com (@VitalikButerin OR @balajis OR @linus_lee) today"
```

- Only keep tweets with **substance** (insights, announcements, paper links) — skip replies, memes, quote dunks
- People who already have RSS (Karpathy, Raschka, Howard, etc.) are covered by Strategy A; only include their tweets if they share something **not on their blog**

### Strategy C — WebSearch (non-RSS, non-Twitter supplement)

Run **one** `WebSearch` call for sources without RSS or active Twitter:

```
query: "Ilya Sutskever Safe Superintelligence news today"
```

- Extract: **title**, **source**, **URL**

## Step 2: Curate

From combined results (Strategy A + B + C), use AI judgment to select the **top 3-5 items** most relevant to the user.

### Priority order

1. AI / LLM breakthroughs (new models, major research, tool releases)
2. Product & startup news (launches, pivots, funding)
3. Deep thinking pieces (strategy, industry analysis)
4. Developer tools & DevEx

### Selection rules

- **Daily quota**: 3-5 items total, never more
- **Dedup**: same story across multiple sources = one item, pick best source
- **Recency**: prefer last 24h, allow up to 48h for low-frequency blogs
- **arXiv**: only surface papers with unusually high engagement or from well-known labs
- **HN**: favor posts with high points-to-time ratio (trending)

### Output format per item

For each selected item, generate:

- **标题**: Chinese translation of the headline (keep proper nouns in English)
- **一句话摘要**: one-sentence Chinese summary of why it matters
- **source**: origin (HN / Karpathy / DeepMind / etc.)
- **link**: original URL

## Step 3: Output to terminal

Print the curated news to the terminal:

```
📰 今日资讯
━━━━━━━━━━
  • {{标题}} — {{一句话摘要}} ({{source}})
    {{link}}
```

This section is **informational only** — no pomodoro estimate, no checkbox.

## Error handling

- If any single source fails (WebFetch timeout, WebSearch error), log briefly and continue with remaining sources.
- If ALL sources fail, print: `⚠️ 所有新闻源获取失败，请检查网络连接。`

## Caller integration

When invoked by another skill (e.g. `/today`), the caller should:

1. Invoke this skill to collect and curate news
2. Take the `📰 今日资讯` output and embed it into their own output format
