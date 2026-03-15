# News Sources Configuration

订阅源配置。修改此文件即可增删源，skill 会自动读取。

## RSS Feeds

### 📚 深度思考与博客

| ID | Name | RSS URL | Category |
|----|------|---------|----------|
| paul-graham | Paul Graham | https://paulgraham.com/rss | thinking |
| stratechery | Stratechery (Ben Thompson) | https://stratechery.com/feed/ | thinking |
| lenny | Lenny's Newsletter | https://lennysnewsletter.com/feed | product |
| acx | Astral Codex Ten (Scott Alexander) | https://astralcodexten.substack.com/feed | thinking |
| joel | Joel on Software | https://joelonsoftware.com/feed/ | thinking |
| ethan-mollick | Ethan Mollick (One Useful Thing) | https://oneusefulthing.org/feed | ai |
| hn | Hacker News (Frontpage 50+) | https://hnrss.org/frontpage?points=50 | tech |

### 🔬 AI 研究与工程

| ID | Name | RSS URL | Category |
|----|------|---------|----------|
| karpathy | Andrej Karpathy | https://karpathy.bearblog.dev/feed/ | ai |
| deepmind | DeepMind Blog | https://deepmind.google/blog/rss.xml | ai |
| google-research | Google Research | https://research.google/blog/rss.xml | ai |
| arxiv-ai | arXiv cs.AI | http://export.arxiv.org/rss/cs.AI | ai |
| langchain | LangChain Blog | https://blog.langchain.dev/rss/ | ai |
| raschka | Sebastian Raschka | https://sebastianraschka.com/rss.xml | ai |
| fastai | fast.ai (Jeremy Howard) | https://www.fast.ai/index.xml | ai |
| distill | Distill.pub | https://distill.pub/rss.xml | ai |

### 🧠 关键人物

| ID | Name | RSS URL | Category |
|----|------|---------|----------|
| sam-altman | Sam Altman | http://blog.samaltman.com/posts.atom | ai |
| dwarkesh | Dwarkesh Patel (Podcast) | https://www.dwarkeshpatel.com/podcast?format=rss | ai |
| amjad | Amjad Masad (Replit CEO) | https://amasad.me/rss | ai |

## Twitter/X Accounts

Twitter 无公开 RSS，通过 WebSearch 抓取近期高互动推文。

### 🔬 核心模型与算法直觉 (The Scientists)

| Handle | Name | Category | Notes |
|--------|------|----------|-------|
| @karpathy | Andrej Karpathy | ai | 也有 RSS，Twitter 补充实时动态 |
| @ilyasut | Ilya Sutskever | ai | 极低频但极高信号 |
| @_akhaliq | AK | ai | 人类版 arXiv RSS，论文速报第一手 |
| @polynoamial | Noam Brown | ai | OpenAI，强化学习/Search/Reasoning |
| @DrJimFan | Jim Fan | ai | NVIDIA，Foundation Agents/具身智能 |

### 🛠️ 产品力与 AI 原生设计 (The Builders)

| Handle | Name | Category | Notes |
|--------|------|----------|-------|
| @jwoodward | Josh Woodward | product | Google Gemini VP，AI 摩擦力思考 |
| @noah_weiss | Noah Weiss | product | Abridge CPO，医疗 AI 垂直落地 |
| @amjad | Amjad Masad | ai | Replit CEO，也有 RSS |
| @lennysan | Lenny Rachitsky | product | 产品增长方法论，也有 RSS |
| @yoheinakajima | Yohei Nakajima | ai | BabyAGI 作者，Autonomous Agents 前沿 |

### ⚙️ 工程架构与 Agent Infra (The Engineers)

| Handle | Name | Category | Notes |
|--------|------|----------|-------|
| @hwchase17 | Harrison Chase | ai | LangChain，Multi-agent Orchestration |
| @rasbt | Sebastian Raschka | ai | 大模型数学→代码可视化，也有 RSS |
| @jeremyphoward | Jeremy Howard | ai | fast.ai，实用主义+算力批判，也有 RSS |
| @karpathy_out | Karpathy Out of Context | ai | Karpathy 各场合工程金句搬运 |

### 🌐 宏观趋势与跨界美学 (The Visionaries)

| Handle | Name | Category | Notes |
|--------|------|----------|-------|
| @dwarkesh_sp | Dwarkesh Patel | thinking | 长篇采访精华片段，也有 RSS |
| @VitalikButerin | Vitalik Buterin | thinking | 去中心化治理/复杂系统，超越 Web3 |
| @balajis | Balaji Srinivasan | thinking | 宏观技术预言，第一性原理 |
| @linus_lee | Linus Lee | product | Notion AI，AI UI/UX 设计灵感 |

## Non-RSS Sources (WebSearch fallback)

People/orgs without RSS or Twitter presence worth tracking separately.

| Name | Tracking Method | Notes |
|------|----------------|-------|
| Ilya Sutskever / SSI | WebSearch "Ilya Sutskever Safe Superintelligence" | Rare but high-signal, supplements @ilyasut |

## Curation Rules

- **Daily quota**: 3–5 items total (not per source)
- **Priority**: ai > product > thinking > tech
- **Recency**: prefer posts from last 24h, allow up to 48h for low-frequency sources
- **Dedup**: same story across multiple sources = one item, pick best source
- **arXiv**: only surface papers with unusually high engagement or from known labs
