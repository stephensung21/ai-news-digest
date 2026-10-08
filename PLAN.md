# AI News Digest: Project Plan

A personal daily or weekly briefing on the latest in AI development, AI agents, evaluation and data science. It pulls from Reddit, Hacker News, arXiv, GitHub and lab blogs, removes duplicates, ranks the results, and summarises them with an LLM.

---

## Overview

- **Goal:** one short digest covering what actually mattered, instead of scrolling a dozen sites.
- **Approach:** a scheduled pipeline (fetch → normalise → dedupe → rank → summarise → deliver) running for free on GitHub Actions.
- **Cost:** a few dollars a month in LLM usage if a cheap model does the per-item summaries.
- **Difficulty:** easy to medium. A useful first version fits in a session or two.

---

## 1. Sources

All of these have free APIs or RSS feeds.

| Source | What to pull | Access |
|---|---|---|
| **Reddit** | r/LocalLLaMA, r/MachineLearning, r/ClaudeAI, r/datascience, r/singularity, r/LangChain, r/OpenAI | Reddit API (register a free "script" app); respect rate limits and the non-commercial terms |
| **Hacker News** | Stories matching AI keywords, with points and comment counts | Algolia HN Search API (no key needed) |
| **arXiv** | New papers in cs.CL, cs.LG, cs.AI, cs.MA | arXiv API / RSS |
| **Hugging Face** | Daily Papers, trending models and spaces | HF API |
| **GitHub** | Trending repos in AI/ML topics | Trending page scrape or GitHub search API |
| **Blogs** | Anthropic, OpenAI, Google DeepMind, Meta AI, Simon Willison, Latent Space, The Batch, Interconnects | RSS |

Keep the source list in a config file (`sources.yaml`) so sources can be added or removed without code changes.

## 2. Pipeline

1. **Fetch** every source on a schedule (for example daily at 6am AEST).
2. **Normalise** into one schema:
   `id, title, url, source, author, score, num_comments, published_at, text_snippet`
3. **Dedupe:** the same story often appears on Reddit, HN and a blog. Group items by canonical URL first, then by title similarity, or by embeddings for near-duplicates. Keep the best-scoring version and list the others as "also discussed on…".
4. **Rank:**
   - Engagement relative to the source's norm (100 upvotes on r/MachineLearning is not the same as 100 on r/singularity).
   - How fast it's rising (score per hour since posting).
   - Source weight (lab announcements weighted higher).
   - Topic match against your interests (agents, evals, data science, and so on).
5. **Summarise and tag** with an LLM:
   - **Per item** (cheap model such as Haiku): a 1–2 sentence summary and topic tags (`agents`, `evals`, `open-weights`, `research`, `tooling`, `industry`), plus a "why it matters" line.
   - **Overview** (stronger model): a short "what mattered this week" summary across the top clusters.
   - Use prompt caching and the Batch API to keep costs down.
6. **Deliver** by one or more of:
   - Email (for example Resend, or Gmail SMTP)
   - A Telegram or Discord bot
   - A static site on GitHub Pages with an archive of past digests
   - Markdown export into an Obsidian vault
7. **Store** every item in SQLite (or a JSON file committed to the repo) so nothing appears twice and the archive is searchable.

## 3. Personalisation (later)

- 👍/👎 or "read" tracking on each item, fed back into the topic and source weights.
- An "interests" profile in config: boosted keywords, muted keywords, muted sources.
- Optional embedding-based recommendations: rank new items by similarity to items you liked.

## 4. Digest format

```
# AI Digest – 8 Oct 2026

## TL;DR
3–5 bullet overview of the most important developments.

## Agents
- **[Title](url)** — summary. *Why it matters:* … (r/LocalLLaMA 1.2k ↑, HN 430 pts)

## Evals & Research
...

## Tools & Releases
...

## Data Science
...
```

## 5. Stack and repository layout

Python 3.12, httpx, feedparser, PRAW (or plain Reddit API calls), SQLite, the Anthropic SDK and Jinja2 templates. GitHub Actions runs the cron job, with API keys stored as repository secrets.

```
ai-news-digest/
  sources.yaml               # sources, subreddits, weights, keywords
  src/fetchers/reddit.py
  src/fetchers/hackernews.py
  src/fetchers/arxiv.py
  src/fetchers/rss.py
  src/fetchers/github.py
  src/normalise.py
  src/dedupe.py
  src/rank.py
  src/summarise.py           # LLM calls: per-item summaries and overview
  src/render.py              # Markdown / HTML templates
  src/deliver.py             # email, Telegram, GitHub Pages
  src/db.py                  # SQLite storage
  templates/digest.md.j2
  tests/
  .github/workflows/daily.yml
```

## 6. Milestones

1. **Reddit and HN fetchers** writing a plain Markdown digest (no LLM yet).
2. **Dedupe and ranking.**
3. **LLM summaries, tags and the overview.**
4. **Delivery channel** (start with email or Telegram).
5. **More sources:** arXiv, Hugging Face, GitHub trending, blog RSS.
6. **GitHub Pages archive.**
7. **Personalisation and feedback loop.**

## 7. Risks and notes

- **Reddit API terms and rate limits:** stay within the free tier, and keep use personal and non-commercial.
- **LLM cost creep:** cap the number of items summarised per run (for example the top 40 after ranking).
- **Summaries can be wrong:** always link to the original, and keep summaries short.
