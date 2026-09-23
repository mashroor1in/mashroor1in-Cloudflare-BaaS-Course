# Lesson 1 — "Cloudflare's Free Tier Explained (2026): What You Actually Get"

**Length target:** 8–10 min | **Type:** Pillar/overview (no coding)

## Hook (0:00–0:15)
- "Cloudflare gives you a serverless backend, a database, file storage, and free AI inference — and most people don't know it exists. By the end of this video you'll know exactly what you get for $0, and by the end of this series you'll have a real AI app running on it."

## Talking points — what to cover
- Show the free-tier table (Workers, Pages, KV, D1, R2, Durable Objects, Queues, Workers AI, Browser Rendering, Analytics Engine).
- Explain the *shape* of Cloudflare's platform: it's not "one free thing," it's a stack — compute (Workers), storage (KV/D1/R2), state (Durable Objects), async (Queues), and AI (Workers AI) all on the same free plan.
- Call out the products that have **no** free tier so nobody wastes time: Vectorize, Hyperdrive, Images, mTLS certificates.
- Explain the single most important mental model: **Workers requests are the shared bottleneck.** Pages Functions, cron triggers, and API calls all draw from the same 100,000 requests/day pool.
- Preview the capstone project (PantryChef) with a quick screen-recorded demo of the finished app — this is what "future you" will have built.
- Set expectations: limits change (give the Sept 2026 D1 enforcement + Workers size bump as a live example of why "last verified" dates matter).

## On-screen graphic — Free Tier Reality Check
| Service | Free limit |
|---|---|
| Workers | 100,000 req/day, 10ms CPU/request, 128MB memory |
| Pages | 500 builds/month, unlimited static bandwidth |
| KV | ~1GB storage, ~100K reads/day, ~1,000 writes/day |
| D1 | 5GB storage, daily row read/write caps (enforced) |
| R2 | 10GB storage, **zero egress fees** |
| Durable Objects | ~400K GB-seconds, ~1M requests/month |
| Queues | Free (added Feb 2026) |
| Workers AI | 10,000 Neurons/day |
| Browser Rendering | 10 min/day |

*(Verify current numbers at developers.cloudflare.com before recording — pin "Last verified: [date]" in the description.)*

## CTA
- "Subscribe and follow along — link to the GitHub repo is in the description, one folder per lesson. Next video: your first Worker live in 5 minutes."
