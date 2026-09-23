# $0 Cloud — "PantryChef" Course Repo

This is the companion repo for the **"$0 Cloud" / "Cloudflare for Free"** YouTube series.
Every lesson lives in its own numbered folder. Each folder has:

- `SCRIPT.md` — hook, concept talking points, step-by-step build notes, and the "Free Tier Reality Check" numbers to show on screen.
- Working code (Worker source, `wrangler.jsonc`, SQL, HTML, etc.) for that lesson.

## The Capstone Project: PantryChef

**PantryChef** is a small AI-powered recipe platform, built one Cloudflare service at a time across the series:

- Users submit and browse recipes (**D1**)
- Trending recipes are cached at the edge (**KV**)
- Recipe photos are uploaded and served from object storage (**R2**)
- A live "Cook-Along" room lets people cook together in real time (**Durable Objects**)
- New recipe submissions trigger background thumbnail + email jobs (**Queues**)
- The submission form is protected from bots and abuse (**Turnstile + rate limiting**)
- A nightly digest email goes out on a schedule (**Cron Triggers**)
- Users can generate a recipe from a list of ingredients using an LLM (**Workers AI**)
- "Ask the Chef" is an AI chatbot with persistent history (**Workers AI + D1**)
- Ingredient-based recipe search works without a paid vector database (**Workers AI embeddings + D1/KV "RAG-lite"**)
- Users can paste a URL and auto-import a recipe from a webpage (**Browser Rendering**)
- Every view and AI generation is tracked for a free analytics dashboard (**Analytics Engine**)
- The whole thing deploys automatically on every push (**GitHub Actions + Wrangler**)

By lesson 19 you'll have one deployed, working, full-stack, AI-enabled app that costs **$0/month** at moderate traffic — and lesson 20 tells viewers exactly when that stops being true.

## Prerequisites (mention in lesson 2)

- Free Cloudflare account
- Node.js 18+
- `npm install -g wrangler` (or use `npx wrangler`)
- `wrangler login`

## Course Map

| # | Folder | Cloudflare service | Builds on |
|---|---|---|---|
| 1 | `01-course-overview` | — | — |
| 2 | `02-first-worker` | Workers | — |
| 3 | `03-pages-landing` | Pages | — |
| 4 | `04-workers-vs-pages` | — (conceptual) | 2, 3 |
| 5 | `05-env-secrets` | Workers config | 2 |
| 6 | `06-d1-database` | D1 | 2 |
| 7 | `07-kv-cache` | KV | 6 |
| 8 | `08-r2-storage` | R2 | 2 |
| 9 | `09-upload-end-to-end` | Pages + Workers + R2 | 3, 8 |
| 10 | `10-durable-objects-cookalong` | Durable Objects | 2 |
| 11 | `11-queues-background-jobs` | Queues | 6 |
| 12 | `12-rate-limiting-turnstile` | Turnstile + rate limiting | 9 |
| 13 | `13-cron-triggers` | Cron Triggers | 6, 7 |
| 14 | `14-workers-ai-intro` | Workers AI | 2 |
| 15 | `15-ai-chatbot` | Workers AI + D1 | 6, 14 |
| 16 | `16-rag-lite-search` | Workers AI embeddings + D1/KV | 6, 7, 14 |
| 17 | `17-browser-rendering-import` | Browser Rendering | 6 |
| 18 | `18-analytics-engine` | Analytics Engine | all |
| 19 | `19-ci-cd-github-actions` | GitHub Actions | all |
| 20 | `20-cost-scaling-breakdown` | — (cost doc) | all |

## Recording order note

Consider resequencing so lesson 14 (Workers AI) is filmed/published earlier (around slot 3–4) since AI content typically drives the highest click-through. The lessons are numbered by *build dependency*, not required publish order — see the pillar course plan doc for the recommended publish sequence.
