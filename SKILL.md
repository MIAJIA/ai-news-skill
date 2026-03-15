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

Using all collected data from Step 1, produce a structured technical briefing.

### Pre-processing

- **Dedup**: same story across multiple sources = one item, pick best source
- **Lab blog auto-include**: posts from OpenAI, Anthropic, DeepMind, Google Research, Meta AI are **always included** — never filtered out
- **Strategy C noise reduction**: if WebSearch only returns old news (> 7 days), skip silently
- **Source attribution**: every item must trace back to a specific account, blog, or URL
- **Fact vs opinion**: distinguish 【事实】(what happened) from 【观点】(someone's judgment)
- **No marketing**: ignore UI updates, promotional content, hype without substance
- **Core judgment**: avoid simple restatement — extract the underlying technical insight

### ① 今日速览（Top Signals）

Only the **5 most important** signals from the past 3 days:
- 重大 AI 模型或产品发布
- 技术路线变化
- 重要行业判断

Each item includes:
- 【事实】发生了什么
- 【观点】发布者或行业的判断
- 【工程含义】这对 AI 系统设计或工程实践意味着什么

**Primary sources**: Lab blogs, HN top posts, high-engagement tweets from lab leaders

### ② AI 公司与开发者发布（Builder's Changelog）

Track:
- 新模型、API 变化、推理性能变化、定价变化、新模态能力、开源模型或权重

Ignore:
- UI 更新、marketing

Analysis focus: 这些发布对 **开发者生态或系统架构** 的实际影响。

**Primary sources**: Lab blogs (OpenAI, Anthropic, DeepMind, Meta AI), LangChain blog, HN

### ③ 工程师 / 研究者观点

Topics: 新架构、agent 系统、reasoning 模型、evaluation 方法、长上下文处理、inference 优化

Requirements: 提炼核心技术判断，不是简单转述。

**Primary sources**: @_akhaliq, @DrJimFan, @ShunyuYao14, @Thom_Wolf, @karpathy, arXiv, Karpathy blog, Raschka blog

### ④ 创业者 / 企业家观点

Topics: AI agent、workflow automation、vertical AI、新应用模式

Analysis focus: 这些判断反映了 **哪些新的产品机会或趋势**。

**Primary sources**: @swyx, @yoheinakajima, @amjad, Lenny, Ethan Mollick, Stratechery

### ⑤ VC / 投资信号

Topics: AI 创业方向、投资热点、技术路径判断

Analysis focus: 不要只记录融资新闻。分析投资叙事反映了市场对哪些技术路径的认可（agent 平台、vertical AI、infra 层、data 层）。

**Primary sources**: @a16z, @sequoia, @benchmark, @foundersfund, @balajis, @deedydas

### ⑥ Architect Decision Lens（架构决策视角）

Synthesized from all sections above. Answer these 4 questions:

1. 今天最值得 reconsider 的技术假设是什么？
2. 哪个技术趋势可能在 6-12 个月内改变 production architecture？
3. 哪个 hype 最可能被证明是 false signal？
4. 如果设计新的 AI 系统，今天的最佳实践会发生什么变化？

### ⑦ One Pattern to Remember

Synthesized from all sections above. Extract one "AI Engineering Pattern" worth remembering.

Format:
- **Pattern**: [one-sentence pattern statement]
- **解释**: 这个模式对未来 AI 系统设计意味着什么

Example: *Reasoning models are shifting evaluation from accuracy → process supervision.*

### Section rules

- Sections with zero relevant items → **omit** (do not show empty headers)
- Every item must have source attribution: `(@handle)`, `(OpenAI Blog)`, `(HN, 350pts)` etc.

## Step 3: Output to terminal

Print the briefing to the terminal.

**Formatting rules:**

- 中文，keep English proper nouns (model names, company names, technical terms)
- Bullet points，高信噪比，信息密度高
- 避免冗长描述，语言像技术 briefing
- No pomodoro estimates, no checkboxes — this is a briefing, not a task list

## Error handling

- If any single source fails (WebFetch timeout, WebSearch error, Twitter rate limit), log briefly and continue with remaining sources.
- If ALL sources fail, print: `⚠️ 所有新闻源获取失败，请检查网络连接。`

## Caller integration

When invoked by another skill (e.g. `/today`), the caller should:

1. Invoke this skill to collect and produce the full briefing
2. Embed the complete output into the `📰 今日资讯` section of the plan
