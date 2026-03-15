# ai-news-skill

A [Claude Code](https://claude.ai/claude-code) skill that generates a high signal-to-noise AI technical briefing from 60+ sources.

One command. 3 minutes. A structured 7-section briefing that tells you what actually matters.

## What it does

Fetches from **3 source types in parallel**:

| Strategy | Sources | Count |
|----------|---------|-------|
| RSS Feeds | OpenAI, DeepMind, Anthropic, HN, LangChain, Karpathy, Stratechery, etc. | 18 feeds |
| Twitter/X | @sama, @DarioAmodei, @karpathy, @swyx, @a16z, etc. | 40+ accounts |
| WebSearch | Sources without RSS or Twitter (e.g., SSI) | fallback |

Outputs a **structured technical briefing** with 7 sections:

| # | Section | What's in it |
|---|---------|-------------|
| 1 | **Top Signals** | 5 most important developments — each with [Fact], [Opinion], [Engineering Implication] |
| 2 | **Builder's Changelog** | New models, API changes, pricing, open-source releases |
| 3 | **Engineer / Researcher Views** | Core technical judgments on architectures, agents, reasoning, evals |
| 4 | **Founder / Entrepreneur Views** | Product opportunities, vertical AI, new application patterns |
| 5 | **VC / Investment Signals** | What investment narratives reveal about technology path validation |
| 6 | **Architect Decision Lens** | 4 questions every system architect should ask today |
| 7 | **One Pattern to Remember** | The single AI engineering pattern worth internalizing |

## Languages

- `SKILL.md` — Chinese (default)
- `SKILL.en.md` — English

Both versions share the same `references/news-sources.md` source configuration.

## Installation

```bash
# Clone the repo
git clone https://github.com/MIAJIA/ai-news-skill.git

# Symlink to Claude Code skills directory
ln -s /path/to/ai-news-skill ~/.claude/skills/news
```

Then run `/news` in Claude Code.

### Optional: Twitter MCP

For higher quality Twitter data, set up the [Twitter MCP server](https://github.com/enescinar/twitter-mcp). Without it, the skill falls back to WebSearch (lower signal).

## File structure

```
ai-news-skill/
├── SKILL.md                    # Skill logic (Chinese)
├── SKILL.en.md                 # Skill logic (English)
├── README.md
└── references/
    └── news-sources.md         # Source configuration (RSS + Twitter + WebSearch)
```

## Customization

Edit `references/news-sources.md` to add or remove sources. The file has 4 sections:

- **RSS Feeds** — add any RSS/Atom URL
- **Twitter/X Accounts** — accounts to track via Twitter MCP or WebSearch
- **Non-RSS Sources** — sources scraped via WebFetch or WebSearch
- **Curation Rules** — priority order, daily quota, dedup rules

## Design decisions

- **Lab blog auto-include**: Posts from OpenAI, Anthropic, DeepMind, Google Research, and Meta AI are always included and never compete with HN for quota
- **7-day window for lab blogs**: These publish infrequently; a 48h window misses most posts
- **Twitter MCP > WebSearch**: `site:x.com` searches mostly return profile pages, not tweets. Twitter MCP returns actual content with engagement metrics
- **2-batch Twitter execution**: 4 parallel Twitter API calls hit rate limits; 2 batches of 2 avoids this
- **Fact/Opinion/Implication structure**: Prevents the briefing from being a link dump

## Contributing

**PRs welcome!** Especially for:

- Adding high-quality information sources to `news-sources.md`
- Improving curation rules
- Adding support for new source types (e.g., Bluesky, Mastodon, YouTube transcripts)

To add a source, edit `references/news-sources.md` and submit a PR.

## License

MIT
