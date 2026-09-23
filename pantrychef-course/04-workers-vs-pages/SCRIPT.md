# Lesson 4 — "Workers vs Pages vs Pages Functions: Which One Do You Actually Need?"

**Length target:** 5–7 min | **Type:** Conceptual (no new code, use lessons 2 & 3 as visuals)

## Hook
- "This is the #1 question I get, and the docs don't make it obvious. Here's the decision tree."

## Talking points / on-screen decision tree
1. **Is it mostly static content (HTML/CSS/JS, a blog, docs, a landing page)?** → Cloudflare Pages.
2. **Does that static site need a *little* server logic (a contact form handler, a redirect, an auth check)?** → Pages Functions (small Workers that live inside your Pages project, in a `functions/` folder).
3. **Is it a full API, a webhook receiver, a scheduled job, or something with no "frontend" at all?** → a standalone Worker.
4. **Important nuance:** Pages Functions requests count against the *same* 100,000 requests/day Workers pool — Pages doesn't give you a second free quota, it shares one.
5. Show the PantryChef architecture diagram: Pages (landing page + frontend) talking to a standalone Worker (API) — this is the pattern used for the rest of the series.

## Free Tier Reality Check
- Static Pages requests: unlimited, free, forever.
- Pages Functions + Worker requests: share one 100,000/day pool.
- This single fact is the most misunderstood part of Cloudflare pricing — worth its own on-screen callout card.

## CTA
"Next: environment variables and secrets, done the right way, before we touch a database."
