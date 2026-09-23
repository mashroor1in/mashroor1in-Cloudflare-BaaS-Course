# Lesson 20 — "When You'll Actually Need to Pay: A Full Cost/Scaling Breakdown"

**Length target:** 10–12 min | **Type:** Wrap-up / conceptual (no new code)

## Hook
- "You've got a full AI app running for $0. Let's talk about exactly what happens when PantryChef actually gets popular."

## Talking points
- Walk through the finished app's architecture diagram one more time, and for each piece, name the specific free-tier ceiling and what upgrading costs:
  - **Workers**: 100K req/day free → Workers Paid is $5/month, includes 10M requests, then $0.30/million.
  - **D1**: daily row read/write caps enforced (Sept 2026) → Paid removes the daily cap, billed per row read/write beyond a higher included amount.
  - **KV**: ~1,000 writes/day free → Paid removes the write cap, billed per operation.
  - **R2**: 10GB free, zero egress either way → Paid is priced per GB stored beyond 10GB, still no egress fee (the biggest ongoing saving vs S3).
  - **Workers AI**: 10,000 Neurons/day free → billed at $0.011/1,000 Neurons beyond that.
  - **Pages**: 500 builds/month → Pro plan ($20/month) raises build concurrency and file-count ceiling.
- Give a concrete "first upgrade" recommendation: almost every project hits the **Workers request cap** before anything else, so that's usually the first $5/month spent, not a whole-platform migration.
- Give viewers a simple rule of thumb: "If you're consistently using more than ~70% of any single free limit for a week straight, budget for that one upgrade — don't pre-pay for headroom you don't need yet."
- Close the series: recap what was built, link the full repo, tease what a "part 2 / paid tier" series could cover if there's demand (gauge audience interest in comments).

## On-screen graphic — Free → Paid at a glance
| Service | Free ceiling | First paid step |
|---|---|---|
| Workers | 100K req/day | $5/mo → 10M req included |
| D1 | Daily row read/write cap | Removed on Paid, usage-billed |
| KV | ~1,000 writes/day | Usage-billed on Paid |
| R2 | 10GB storage | ~$0.015/GB/month beyond 10GB, egress still free |
| Workers AI | 10,000 Neurons/day | $0.011/1,000 Neurons beyond |
| Pages | 500 builds/month | Pro plan, $20/month flat |

*(Re-verify every number against current Cloudflare pricing docs immediately before recording — this table ages the fastest of any video in the series.)*

## CTA
- "That's the whole series — a full AI app, for $0, top to bottom. Repo's linked below, every lesson its own folder. If you want a part 2 on scaling this into a real paid product, let me know in the comments."
