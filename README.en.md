# AIHOT — a daily hot-topics site for your industry

> Swap in your own sources and selection standards. It filters every day and publishes a daily briefing for your field.

[简体中文](README.md) | English

<p><img src="docs/assets/board-light.png" width="900" alt="AIHOT demo site: the model leaderboard page"></p>

<p><img src="docs/assets/cluster-light.png" width="900" alt="Event grouping and heat: several reports become one event, with the current hot list on the right"></p>

<sub>The leaderboard is a real screenshot from the local demo site; in the second image the left half illustrates grouping and the right half is the live hot list. Default site name: MyHOT.</sub>

## What problem it solves

Every industry produces more news than anyone can read, and only a few items are worth reading. Watching dozens of sources by hand is not realistic, while reposting everything drowns readers in marketing copy and duplicate coverage.

AIHOT turns this into a configurable pipeline: collect from sources, decide whether an item belongs to the field, score it twice independently against the same rubric, and keep it only if it clears a tier-specific threshold. It then writes Chinese headlines and summaries, clusters reports of the same story into one event, ranks events by the number of independent sources, and publishes daily, weekly and monthly briefings on schedule.

The live site is [aihot.news](https://aihot.news), serving the AI field. This repository is its complete framework, including every prompt and threshold; replace the industry configuration in `industry/` and you have a site for another field.

## Core capabilities

- **Six source types**: RSS, web lists, JSON APIs, X accounts, WeChat public accounts, and external pushes from your own scripts. Sources have tiers (first-party official vs. media/individual), and polling frequency adjusts to output.
- **Calibratable selection**: after a prefilter, the same rubric scores each item twice; the two scores must sum to at least 2 × threshold. Thresholds differ by source tier. Prompts and thresholds live in `industry/`; label your own sample, run `scripts/eval-selection.ts`, and compare runs in the admin SelectBench.
- **Writing**: Chinese headlines, answer-first summaries, reasons for selection, tags, and translation of foreign-language material; guards keep the model from naming companies absent from the original.
- **Event grouping**: candidate events are found by title/summary vectors, then a model decides whether reports are the same story, a follow-up, or two different stories; uncertain merges are confirmed by a second model. Manual assignments are never overwritten.
- **Heat**: computed per event; within 48 hours each independent source counts once and a 24-hour half-life applies. Compared with 6 hours ago, rising events are marked as trending and new ones as new.
- **Daily / weekly / monthly briefings**: daily at 08:00, weekly on Mondays at 10:00, monthly on the 1st at 10:30, sectioned by category.
- **Outputs for people and agents**: web pages, RSS (curated / all / full text / daily), public API `/api/v1/`, MCP `/api/mcp`, `llms.txt`, sitemap and share images, all through one public reading layer.
- **Admin**: source management with test fetches, content diagnostics, selection evaluation, per-step model choice, budget cut-offs for paid services, run logs and alerts.
- **Optional AI-only modules**: the model leaderboard and Codex reset monitor; set them to `false` in `industry/features.ts` to remove their pages and scheduled jobs.

## Quick start

Requirements: Docker with Compose, Node.js 24.11 or newer (only to generate `.env`), and an OpenAI-compatible model API key.

```bash
node scripts/init-env.ts --llm-key <your model API key>
docker compose up -d --build
```

`init-env.ts` creates `.env` with random secrets and an admin password, printing the password once (it is also stored in `ADMIN_PASSWORD`). Without Node, copy `.env.example` to `.env` and fill in `ADMIN_PASSWORD` (at least 12 characters), `SESSION_SECRET`, `IMG_PROXY_SIGN_SECRET`, `POSTGRES_PASSWORD` and `LLM_API_KEY` yourself.

Open <http://localhost:3000>; the admin is at `/admin`. The first start imports the demo sources, content appears within a minute or two, and the first batch of roughly a hundred items takes about half an hour to process.

`docker compose` starts five containers: `db` (PostgreSQL 17), `setup` (runs migrations and seeds, then exits), `api`, `worker` and `web`. Before deploying to a server, set `SITE_URL` in `.env` to the address readers actually use; otherwise RSS, the sitemap and share links keep pointing at localhost.

To adapt it to your field, change the site copy, categories, demo sources, prompts and thresholds in [`industry/`](industry/); the steps are in [Customize it for your industry](docs/customize.md). Afterwards you can run:

```bash
npm run typecheck
npm run test
node scripts/smoke.ts
```

## Data and privacy

- Content, events and briefings live in PostgreSQL 17; uploaded files and local data live in `AIHOT_DATA_DIR` (`/data` in Docker).
- Readers do not need an account. Their favorites and read state stay in the browser; the admin area is restricted to administrators.
- Every selected item links to the original. Whether the full text is shown on-site is controlled by the source's `site_fulltext`; the default is summary only.
- `.env` holds the admin password, database password and API keys, and is excluded by `.gitignore`; `.data/` is also kept out of Git.
- Paid calls (models, X, WeChat, Jina) record a receipt before returning a result, and retries reuse already-paid results. Each service has per-minute, hourly and daily budget ceilings that pause it when exceeded.

## Known limitations

- This is a snapshot of the code running at `aihot.news`, not a polished general-purpose framework; upstream updates are not guaranteed to be synced every time.
- The repository ships only public overseas AI news demo sources. It does not include AIHOT's production source list or operational data.
- X sources need a SocialData key, WeChat public accounts need a Dajiala key, and optional web rendering uses Jina; all are billed per request. You also need your own OpenAI-compatible model key.
- Docs and UI are Simplified Chinese only; no English site UI is provided in this repository.
- Servers in mainland China need their own ICP filing, with the number placed in `industry/site.ts`. If overseas sources are unreachable, set `EGRESS_PROXY_URL` in `.env`.
- The model leaderboard and Codex reset monitor are on by default; non-AI fields should disable them in `industry/features.ts`.

## License

Code is under the [MIT License](LICENSE). The AIHOT name and logo are not covered by the license. Fonts, model vendors and evaluation sources keep their own licenses and trademarks; see [NOTICE](NOTICE).
