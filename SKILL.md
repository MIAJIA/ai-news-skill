---
name: news
description: 高信噪比 AI 技术简报。从 RSS、Twitter/X、WebSearch 并行采集过去 3 天动态，结构化分析输出 7 section 技术 briefing（Top Signals、Builder's Changelog、研究者观点、创业者观点、VC 信号、架构决策视角、Pattern）。可独立使用或被 /today 调用。
---

# /news

高信噪比 AI 技术简报。并行采集 → 结构化分析 → 终端输出 7 section 技术 briefing。

## Step 1: Fetch sources

Read source configuration from `references/news-sources.md` (relative to this skill's base directory).

Run **all three strategies in parallel**.

### Strategy A — RSS Feeds (primary, low-token)

Read all RSS URLs from `references/news-sources.md` (the "RSS Feeds" tables).

Use `WebFetch` to fetch all feeds **in parallel**, batched up to 5 concurrent calls:

- Batch 1: OpenAI, Anthropic, Meta AI, DeepMind, Google Research
- Batch 2: HN, Karpathy, Ethan Mollick, LangChain, arXiv cs.AI
- Batch 3: Stratechery, Lenny, Paul Graham, Astral Codex Ten, Joel on Software
- Batch 4: Sebastian Raschka, fast.ai, Distill.pub, Sam Altman, Dwarkesh Patel, Amjad Masad

For each feed:

- **Lab blogs** (OpenAI, Anthropic, Meta AI, DeepMind, Google Research): extract titles and dates from the last **7 days**
- **All other feeds**: extract titles and dates from the last **3 days**
- Do NOT fetch full article content (saves tokens)
- If a feed fails, skip silently and continue

### Strategy B — Twitter/X (high-signal accounts)

Fetch tweets from key accounts listed in `references/news-sources.md` (the "Twitter/X Accounts" tables).

**B1. Twitter MCP (preferred — higher signal)**

If `mcp__twitter__search_tweets` is available, use it as the primary Twitter source.

Run `mcp__twitter__search_tweets` calls (count: 20 each) **sequentially in two batches** to avoid rate limits:

**Batch 1** (run these two in parallel):

1. AI Lab Leaders:

```
query: "from:sama OR from:DarioAmodei OR from:demishassabis OR from:gdb OR from:geoffreyhinton"
```

2. AI Researchers + Engineers:

```
query: "from:_akhaliq OR from:DrJimFan OR from:polynoamial OR from:ShunyuYao14 OR from:Thom_Wolf"
```

**Batch 2** (run after Batch 1 completes):

3. Builders + Anthropic:

```
query: "from:claudeai OR from:alexalbert__ OR from:AmandaAskell OR from:swyx OR from:yoheinakajima OR from:deedydas"
```

4. Visionaries + VCs:

```
query: "from:VitalikButerin OR from:balajis OR from:elonmusk OR from:a16z OR from:sequoia OR from:foundersfund"
```

From the results, filter to tweets from the last 3 days only.

**Rate limit handling**: If a call returns a rate limit error, wait 2 seconds and retry once. If it fails again, skip that group and continue — partial Twitter data is better than none.

**B2. WebSearch fallback**

If Twitter MCP is not available (tool not found or connection error), fall back to `WebSearch` with `site:x.com` queries using the same account groupings:

1. `"site:x.com (@sama OR @DarioAmodei OR @demishassabis OR @gdb OR @geoffreyhinton) today"`
2. `"site:x.com (@_akhaliq OR @DrJimFan OR @polynoamial OR @ShunyuYao14 OR @Thom_Wolf) today"`
3. `"site:x.com (@claudeai OR @alexalbert__ OR @AmandaAskell OR @swyx OR @yoheinakajima OR @deedydas) today"`
4. `"site:x.com (@VitalikButerin OR @balajis OR @elonmusk OR @a16z OR @sequoia OR @foundersfund) today"`

**Filtering rules (both B1 and B2):**

- Only keep tweets with **substance** (insights, announcements, paper links) — skip replies, memes, quote dunks
- People who already have RSS (Karpathy, Raschka, Howard, etc.) are covered by Strategy A; only include their tweets if they share something **not on their blog**

### Strategy C — WebSearch (non-RSS, non-Twitter supplement)

Run **one** `WebSearch` call for sources without RSS or active Twitter:

```
query: "Ilya Sutskever Safe Superintelligence news today"
```

- Extract: **title**, **source**, **URL**

## Step 2: Curate and analyze

Read the briefing prompt from `references/briefing-prompt.md`.

Using all collected data from Step 1, apply the briefing prompt to produce a structured 7-section technical briefing.

### Pre-processing rules

Before applying the briefing prompt:

- **Dedup**: same story across multiple sources = one item, pick best source
- **Lab blog auto-include**: posts from OpenAI, Anthropic, DeepMind, Google Research, Meta AI are **always included** in the relevant section — they are never filtered out
- **Strategy C noise reduction**: if WebSearch only returns old news (> 7 days), skip silently
- **Source attribution**: every item must trace back to a specific account, blog, or URL

### Section-to-source mapping

Guide for populating each section:

| Section | Primary sources |
|---------|----------------|
| ① Top Signals | Lab blogs, HN top posts, high-engagement tweets from lab leaders |
| ② Builder's Changelog | Lab blogs (OpenAI, Anthropic, DeepMind, Meta AI), LangChain blog, HN |
| ③ Engineer/Researcher views | @_akhaliq, @DrJimFan, @ShunyuYao14, @Thom_Wolf, @karpathy, arXiv, Karpathy blog, Raschka blog |
| ④ Founder/Entrepreneur views | @swyx, @yoheinakajima, @amjad, Lenny, Ethan Mollick, Stratechery |
| ⑤ VC signals | @a16z, @sequoia, @benchmark, @foundersfund, @balajis, @deedydas |
| ⑥ Architect Decision Lens | Synthesized from all sections above |
| ⑦ One Pattern | Synthesized from all sections above |

Sections with zero relevant items should be **omitted** (do not show empty sections).

## Step 3: Output to terminal

Print the briefing following the structure defined in `references/briefing-prompt.md`.

**Formatting rules:**

- Language: 中文, keep English proper nouns (model names, company names, technical terms)
- Bullet points, high information density
- Every item has source attribution: `(@handle)`, `(OpenAI Blog)`, `(HN, 350pts)` etc.
- No pomodoro estimates, no checkboxes — this is a briefing, not a task list

## Error handling

- If any single source fails (WebFetch timeout, WebSearch error, Twitter rate limit), log briefly and continue with remaining sources.
- If ALL sources fail, print: `⚠️ 所有新闻源获取失败，请检查网络连接。`

## Caller integration

When invoked by another skill (e.g. `/today`), the caller should:

1. Invoke this skill to collect and produce the full briefing
2. Embed the complete output into the `📰 今日资讯` section of the plan
