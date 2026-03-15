---
name: news
description: 高信噪比 AI 技术简报。从 RSS、Twitter/X、WebSearch 并行采集过去 3 天动态，结构化分析输出 7 个板块的技术简报（今日速览、开发者发布、研究者观点、创业者观点、VC 信号、架构决策视角、值得记住的模式）。可独立使用或被 /today 调用。
---

# /news

高信噪比 AI 技术简报。并行采集 → 结构化分析 → 终端输出 7 个板块的技术简报。

## 第一步：采集数据源

读取 `references/news-sources.md`（相对于本 skill 的根目录）中的源配置。

**三条策略并行执行。**

### 策略 A — RSS 订阅（主力，低 token 消耗）

从 `references/news-sources.md` 的 "RSS Feeds" 表格中读取所有 RSS URL。

用 `WebFetch` **并行**拉取所有 feed，每批最多 5 个：

- 第 1 批：OpenAI、Meta AI、DeepMind、Google Research
- 第 2 批：HN、Karpathy、Ethan Mollick、LangChain、arXiv cs.AI
- 第 3 批：Stratechery、Lenny、Paul Graham、Astral Codex Ten、Joel on Software
- 第 4 批：Sebastian Raschka、fast.ai、Distill.pub、Sam Altman、Dwarkesh Patel、Amjad Masad

每个 feed 的处理规则：

- **实验室博客**（OpenAI、Meta AI、DeepMind、Google Research）：提取最近 **7 天**的标题和日期
- **其他 feed**：提取最近 **3 天**的标题和日期
- 不要拉取文章全文（节省 token）
- 某个 feed 失败时，静默跳过，继续处理其他源

**Anthropic（无 RSS — 直接抓取 HTML 页面）**

Anthropic 没有 RSS feed。用 `WebFetch` **并行**抓取以下两个页面（可与 RSS 第 1 批同时执行）：

1. `https://www.anthropic.com/engineering` — 提取最近 7 天的文章标题、日期、URL
2. `https://www.anthropic.com/research` — 提取最近 7 天的文章标题、日期、URL

这些属于实验室博客，遵循与其他实验室博客相同的自动收录规则。

### 策略 B — Twitter/X（高信噪比账号）

从 `references/news-sources.md` 的 "Twitter/X Accounts" 表格中获取要追踪的账号列表。

**B1. Twitter MCP（首选 — 信号质量更高）**

如果 `mcp__twitter__search_tweets` 可用，优先使用它作为 Twitter 数据源。

执行 `mcp__twitter__search_tweets` 调用（每次 count: 20），**分两批串行执行**以避免触发速率限制：

**第 1 批**（这两个并行执行）：

1. AI 实验室负责人：

```
query: "from:sama OR from:DarioAmodei OR from:demishassabis OR from:gdb OR from:geoffreyhinton"
```

2. AI 研究者与工程师：

```
query: "from:_akhaliq OR from:DrJimFan OR from:polynoamial OR from:ShunyuYao14 OR from:Thom_Wolf"
```

**第 2 批**（等第 1 批完成后执行）：

3. 构建者与 Anthropic：

```
query: "from:claudeai OR from:alexalbert__ OR from:AmandaAskell OR from:swyx OR from:yoheinakajima OR from:deedydas"
```

4. 远见者与 VC：

```
query: "from:VitalikButerin OR from:balajis OR from:elonmusk OR from:a16z OR from:sequoia OR from:foundersfund"
```

从结果中只保留最近 3 天的推文。

**速率限制处理**：如果某次调用返回速率限制错误，等待 2 秒后重试一次。如果仍然失败，跳过该组继续执行 — 部分 Twitter 数据好过没有数据。

**B2. WebSearch 降级方案**

如果 Twitter MCP 不可用（工具未找到或连接错误），降级为使用 `WebSearch` 的 `site:x.com` 查询，使用相同的账号分组：

1. `"site:x.com (@sama OR @DarioAmodei OR @demishassabis OR @gdb OR @geoffreyhinton) today"`
2. `"site:x.com (@_akhaliq OR @DrJimFan OR @polynoamial OR @ShunyuYao14 OR @Thom_Wolf) today"`
3. `"site:x.com (@claudeai OR @alexalbert__ OR @AmandaAskell OR @swyx OR @yoheinakajima OR @deedydas) today"`
4. `"site:x.com (@VitalikButerin OR @balajis OR @elonmusk OR @a16z OR @sequoia OR @foundersfund) today"`

**过滤规则（B1 和 B2 通用）：**

- 只保留有**实质内容**的推文（洞察、公告、论文链接）— 跳过回复、meme、抬杠
- 已有 RSS 的人（Karpathy、Raschka、Howard 等）由策略 A 覆盖；仅当他们在 Twitter 上分享了**博客中没有的内容**时才收录

### 策略 C — WebSearch（无 RSS、无 Twitter 的补充源）

对没有 RSS 也没有活跃 Twitter 的源，执行**一次** `WebSearch` 调用：

```
query: "Ilya Sutskever Safe Superintelligence news today"
```

- 提取：**标题**、**来源**、**URL**

## 第二步：筛选与分析

使用第一步采集的所有数据，生成结构化技术简报。

### 预处理规则

- **去重**：同一条新闻出现在多个源中 = 合并为一条，选最佳来源
- **实验室博客自动收录**：来自 OpenAI、Anthropic、DeepMind、Google Research、Meta AI 的文章**必须收录** — 永远不会被过滤掉
- **策略 C 降噪**：如果 WebSearch 只返回旧闻（> 7 天），静默跳过
- **来源标注**：每条信息必须追溯到具体的账号、博客或 URL
- **区分事实与观点**：区分【事实】（发生了什么）和【观点】（某人的判断）
- **过滤营销内容**：忽略 UI 更新、推广内容、无实质的炒作
- **提炼核心判断**：避免简单转述 — 提取底层的技术洞察

### ① 今日速览

过去 3 天中**最重要的 5 条**动态：
- 重大 AI 模型或产品发布
- 技术路线变化
- 重要行业判断

每条包含：
- 【事实】发生了什么
- 【观点】发布者或行业的判断
- 【工程含义】这对 AI 系统设计或工程实践意味着什么

**主要数据源**：实验室博客、HN 高分帖、实验室负责人的高互动推文

### ② AI 公司与开发者发布

追踪内容：
- 新模型、API 变化、推理性能变化、定价变化、新模态能力、开源模型或权重

忽略内容：
- UI 更新、营销内容

分析重点：这些发布对**开发者生态或系统架构**的实际影响。

**主要数据源**：实验室博客（OpenAI、Anthropic、DeepMind、Meta AI）、LangChain 博客、HN

### ③ 工程师 / 研究者观点

关注话题：新架构、agent 系统、推理模型、评估方法、长上下文处理、推理优化

要求：提炼核心技术判断，不是简单转述。

**主要数据源**：@_akhaliq、@DrJimFan、@ShunyuYao14、@Thom_Wolf、@karpathy、arXiv、Karpathy 博客、Raschka 博客

### ④ 创业者 / 企业家观点

关注话题：AI agent、工作流自动化、垂直 AI、新应用模式

分析重点：这些判断反映了**哪些新的产品机会或趋势**。

**主要数据源**：@swyx、@yoheinakajima、@amjad、Lenny、Ethan Mollick、Stratechery

### ⑤ VC / 投资信号

关注话题：AI 创业方向、投资热点、技术路径判断

分析重点：不要只记录融资新闻。分析投资叙事反映了市场对哪些技术路径的认可（agent 平台、垂直 AI、基础设施层、数据层）。

**主要数据源**：@a16z、@sequoia、@benchmark、@foundersfund、@balajis、@deedydas

### ⑥ 架构决策视角

综合以上所有板块，回答以下 4 个问题：

1. 今天最值得重新审视的技术假设是什么？
2. 哪个技术趋势可能在 6-12 个月内改变生产架构？
3. 哪个热点最可能被证明是虚假信号？
4. 如果今天设计新的 AI 系统，最佳实践会发生什么变化？

### ⑦ 值得记住的模式

综合以上所有板块，提炼一个最值得记住的 "AI 工程模式"。

格式：
- **模式**：[一句话模式描述]
- **解释**：这个模式对未来 AI 系统设计意味着什么

示例：*推理模型正在将评估标准从准确率转向过程监督。*

### 板块规则

- 没有相关内容的板块 → **省略**（不显示空标题）
- 每条信息必须标注来源：`(@handle)`、`(OpenAI 博客)`、`(HN, 350分)` 等

## 第三步：终端输出

将简报打印到终端。

**格式要求：**

- 中文，保留英文专有名词（模型名、公司名、技术术语）
- 要点式排版，高信噪比，高信息密度
- 避免冗长描述，语言风格像技术简报
- 不要番茄钟估时，不要复选框 — 这是简报，不是任务清单

## 错误处理

- 如果单个数据源失败（WebFetch 超时、WebSearch 错误、Twitter 速率限制），简要记录后继续处理其他源。
- 如果所有数据源都失败，打印：`⚠️ 所有新闻源获取失败，请检查网络连接。`

## 调用方集成

当被其他 skill（如 `/today`）调用时，调用方应：

1. 调用本 skill 采集并生成完整简报
2. 将完整输出嵌入到计划的 `📰 今日资讯` 板块中
