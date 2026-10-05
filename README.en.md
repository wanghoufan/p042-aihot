# AIHOT — a hot-topics daily site you can rebrand to any industry

[简体中文](./README.md) | English

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="docs/assets/banner-dark.png">
    <img src="docs/assets/banner-light.png" alt="AIHOT: every industry can have its own AIHOT. Many source streams flow into one curated selection in the middle, which is then distributed to law, HR, finance, and other industries" width="100%">
  </picture>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-176b75?style=flat-square" alt="MIT License"></a>
  <img src="https://img.shields.io/badge/Node.js-24-176b75?style=flat-square&logo=nodedotjs&logoColor=white" alt="Node.js 24">
  <img src="https://img.shields.io/badge/PostgreSQL-17-176b75?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL 17">
  <img src="https://img.shields.io/badge/Docker-Compose-176b75?style=flat-square&logo=docker&logoColor=white" alt="Docker Compose">
  <a href="https://aihot.news"><img src="https://img.shields.io/badge/demo-aihot.news-202a30?style=flat-square" alt="aihot.news"></a>
</p>

<p align="center">
  <b>A website framework that finds its own hot topics and writes its own daily briefing.</b><br>
  Swap in your own sources, swap in your own KnowHow for what counts as worth reading, and it becomes your industry's hot-topics site.
</p>

[Quick start](#quick-start) · [Customize it](docs/customize.md) · [How it works](#how-it-works) · [Documentation](#documentation) · [Community](https://github.com/KKKKhazix/AIHOT/discussions)

## What is this

[AIHOT](https://aihot.news) is an AI news site I built. Every day it collects material from a set of sources, has a large language model filter it first and then score it twice independently, keeps only what is genuinely worth reading, and writes Chinese headlines and summaries for it; reports from different outlets about the same story are clustered into one event, events are ranked by how many people are talking about them, and a daily briefing is published every morning.

This repository is the complete framework: the site, the admin panel, the selection pipeline, the clustering and heat algorithms, **the full text of every prompt and every selection threshold**.

## Why open source

Over the past half year, many friends in law, HR, finance, and precious metals have asked me to build the same thing for their field.

I can't. I don't know your industry — which sources are useful, or what even counts as "hot" for you.

But you do.

Since I can't serve everyone, I'd rather hand the spark to you.

## Read this first

- **I am not a professional developer.** I come from design and, half a year ago, could barely read code. This codebase was rewritten with AI and is much cleaner than before, but parts of it are certainly still poorly written. Issues are welcome; I may not reply quickly, and I apologize in advance.
- **This is a snapshot.** It comes from the code AIHOT is running in production — not a polished general-purpose framework. I will try to sync future AIHOT updates into this repo, but I cannot promise every one.
- **It does not contain AIHOT's real source list or operational data.** The repo ships 18 public overseas AI news feeds as a demo — enough to run it and see the effect. The real sources must be replaced with ones from your own industry.
- **Please do not use the name AIHOT or its logo.** Put your own name on it, and it is your site.

## Quick start

To create your own standalone site, you can start from [Use this template](https://github.com/KKKKhazix/AIHOT/generate) and then clone the repository it generates. If you plan to keep merging upstream updates or contribute code, forking first is recommended. The commands below are for a direct trial run.

You need [Docker](https://docs.docker.com/get-docker/) and an OpenAI-compatible model API key (DeepSeek, Qwen, or Zhipu all work).

```bash
git clone https://github.com/KKKKhazix/AIHOT.git myhot
cd myhot
node scripts/init-env.ts --llm-key <your-model-api-key>
docker compose up -d --build
```

Open <http://localhost:3000>. The admin panel is at `/admin`; the administrator password is in `ADMIN_PASSWORD` in `.env`. Content starts appearing within a minute or two, and the first import takes roughly half an hour to process.

If your server has no Node installed, is in mainland China, or you need a domain and HTTPS, see [Deployment](docs/deploy.md).

## How it works

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/how-dark.png">
  <img src="docs/assets/how-light.png" alt="Six steps: collect, pre-filter, score twice, write, cluster, rank and publish" width="100%">
</picture>

An item enters from a source, is de-duplicated, then pre-filtered; anything that might matter is scored twice independently and only enters the selection if it passes the threshold; then a Chinese headline and summary are written, the item is clustered with other reports into an event, counted into the heat score, and finally lands in the daily briefing. The prompt for each step lives in [`industry/prompts/`](industry/prompts/) — changing the standard never requires touching code. Details in [Selection and calibration](docs/selection.md).

### Clustering and hot topics

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/cluster-dark.png">
  <img src="docs/assets/cluster-light.png" alt="Reports from five sources clustered into one event, the event entering the current hot list" width="100%">
</picture>

The same story — one post on an official site, ten reprints in media, a day of arguing on X — should be seen only once by a reader. AIHOT clusters them into one **event**: it first uses title-and-summary vectors to find candidates within the past two weeks, then asks a model whether they are the same event, a follow-up development, or two different things; uncertain cases are merged and re-checked by a different model.

**Heat** is computed per event, not per article: within 48 hours, each independent source counts only once, and its contribution halves after 24 hours. Re-fetching never inflates a score, and one outlet publishing ten pieces still counts once — so what ranks at the top is what a lot of people are actually talking about.

### Speed

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/perf-dark.png">
  <img src="docs/assets/perf-light.png" alt="Live AIHOT measurements: page median 10 ms, 95th percentile within 50 ms; API median 6 ms, 95th percentile within 12 ms; article pages 95th percentile within 14 ms" width="100%">
</picture>

## What you get

| | |
|---|---|
| **Six source types** | RSS, web page lists, JSON APIs, X accounts, WeChat Official Accounts, and content pushed in by your own scripts. Sources are graded (official first-hand / media and individuals), and crawl frequency adjusts automatically to output |
| **Selection** | Pre-filtering, the same scoring rubric applied twice independently, then an acceptance threshold that depends on the source grade. Prompts and thresholds are fully public and fully editable; calibrate with your own labeled samples in SelectBench |
| **Writing** | Chinese headlines, answer-first summaries, recommendation reasons, tags, full translation of foreign-language text; a guard stops the model from putting a company into a headline when the original never mentioned it |
| **Clustering** | Reports of the same story become one event, follow-up developments hang under it, and event pages carry a synopsis; developments and the reporting timeline can be toggled together between newest-first and earliest-first; manual re-assignments are never overwritten |
| **Hot topics** | Heat is computed per event: more independent sources ranks higher, and X discussions count too; events are compared with 6 hours ago — fast risers get an upward marker, newcomers get a "new" marker |
| **Daily / weekly / monthly briefings** | Daily at 08:00, weekly on Mondays, monthly on the 1st; organized by category with an opening summary |
| **Topics and search** | Three kinds of topic pages (companies, directions, content formats); search over titles and summaries plus related search over full text |
| **Agent interfaces** | RSS (selected / all / full text / daily), a public API, MCP (Model Context Protocol, a way to plug data into AI agents), Agent Markdown, and `llms.txt` — one body of content served to both humans and agents |
| **Admin panel** | Source management and test fetching, content diagnostics, selection benchmarking, a different model per pipeline step, budget cutoff for paid services, run logs and alerts |
| **AI-specific modules** | The model leaderboard (aggregating several public benchmarks with a published methodology) and Codex reset monitoring. Other industries can switch both off with one toggle |

## A look

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/shots-dark.png">
  <img src="docs/assets/shots-light.png" alt="Homepage with current hot topics and the daily selection, and the sources-river diagram on the about page" width="100%">
</picture>

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="docs/assets/board-dark.png">
  <img src="docs/assets/board-light.png" alt="The model leaderboard" width="100%">
</picture>

<p align="center"><sub>Screenshots come from a local instance running on the demo sources; the site name shown is the default MyHOT.</sub></p>

## Turn it into your industry

After deployment, open `/agent` to copy the integration snippets for MCP, RSS, API, or Markdown. Agents that can only read web pages can view the discovery instructions at `/api/v1/agent` and then read the selected feed, search, hot topics, events, and daily briefing as Markdown; structured JSON for weeklies and monthlies is discovered at `/api/v1/weeklies` and `/api/v1/monthlies` — append `/latest` or one issue's ISO week/month key to read it. Structured data lives under `/api/v1/`, and the API contract at `/openapi-v1.json`. All these outlets honor article withdrawals and full-text permissions, and take the site name, links, and categories from your industry configuration.

The easiest way: open your coding agent (Claude Code, Codex, or similar), hand it this repository, and say:

```text
Read AGENTS.md and docs/customize.md, and turn this site into a hot-topics site
for the "legal" industry. What I care about is: … (which sources you want to watch,
what makes news important or unimportant for you — the more specific, the better).
```

Almost everything you need to change lives in the single [`industry/`](industry/) folder; the code barely changes:

| File | What to change |
|---|---|
| `site.ts` | Site name, industry wording, homepage copy, about page |
| `taxonomy.ts`, `topics.json` | Categories, tags, topics |
| `sources.json` | Sources imported on first start |
| `prompts/` | Selection standards and writing requirements. **Your industry's KnowHow goes here** |
| `selection.ts` | Acceptance thresholds |
| `features.ts` | Toggles for the model leaderboard and Codex reset monitoring |
| `brand/`, `pages/` | Icons, usage rules, privacy notice |

The time worth spending most is on the scoring rubric (`prompts/selection-score.md`) and the thresholds: take 100–200 items you have labeled yourself, run `scripts/eval-selection.ts`, see how accurately it selects, then go back and revise. How to do this is written up in [Selection and calibration](docs/selection.md).

## Documentation

All linked documents are currently in Chinese.

| Document | Contents |
|---|---|
| [Turn it into your industry](docs/customize.md) | Site name, categories, sources, prompts, thresholds, branding, step by step |
| [Sources](docs/sources.md) | How to configure the six source types, grading and full text, the external push API |
| [Selection and calibration](docs/selection.md) | How an item becomes selected, how to calibrate with your own samples |
| [Event grouping and relationship benchmarking](docs/grouping.md) | How event relationships are judged, how to benchmark with your own pairwise gold set |
| [Deployment](docs/deploy.md) | Docker, domain and HTTPS, mainland China, updates, backups, what it costs |
| [Architecture](docs/architecture.md) | The three processes, a few invariants, directory layout, external outlets |
| [Model leaderboard and Codex reset monitoring](docs/leaderboard.md) | The two AI-specific modules |

Tech stack: Node.js 24 · TypeScript · React Router (server rendering) · Fastify · PostgreSQL · pg-boss · Tailwind CSS · Docker Compose.

## Community and contributing

For deployment and usage questions use the [Q&A board](https://github.com/KKKKhazix/AIHOT/discussions/categories/q-a); for new ideas, the [ideas board](https://github.com/KKKKhazix/AIHOT/discussions/categories/ideas); and you are welcome to show the industry hot-topics site you built on the [show-and-tell board](https://github.com/KKKKhazix/AIHOT/discussions/categories/show-and-tell).

Found a bug or have a concrete feature suggestion? [Open an Issue](https://github.com/KKKKhazix/AIHOT/issues/new/choose). Before changing code, read the [contributing guide](CONTRIBUTING.md); report security vulnerabilities through the [private disclosure channel](SECURITY.md).

## Finally

AIHOT began as a very, very small idea on one of my countless late nights.

I don't know what it will be turned into or how far it will go. But that may be the most romantic part of open source.

The rest of the road is yours.

<p align="right">—— 数字生命卡兹克 (Kazik)</p>

## License

The code is under the [MIT License](LICENSE). The AIHOT name and logo are **not** part of the license. Fonts, model vendors, and benchmark sources each have their own licenses and trademark ownership — see [NOTICE](NOTICE).
