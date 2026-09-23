# Lesson 13 — "Cron Triggers: Scheduled Tasks Without a Server"

**Length target:** 6–8 min | **Type:** Hands-on

## Hook
- "No server means no crontab. Here's how scheduled jobs work on Cloudflare — a nightly trending-recipes digest, built in a few lines."

## Talking points
- `triggers.crons` in `wrangler.jsonc` — standard cron syntax, runs your Worker's `scheduled()` handler.
- Local testing: `wrangler dev --test-scheduled` to trigger it manually without waiting for the real schedule.
- PantryChef use case: every night, query D1 for the top 5 recipes by view count (ties into lesson 18's analytics) and log/email a digest.

## Build steps
1. Add a cron trigger to `wrangler.jsonc`.
2. Implement `scheduled()` alongside the existing `fetch()` export.
3. Test with `wrangler dev --test-scheduled`, then deploy and check the dashboard's Cron Triggers tab for run history.

## Code — wrangler.jsonc (add)
```jsonc
{
  "triggers": {
    "crons": ["0 6 * * *"]
  }
}
```

## Code — src/index.ts (add scheduled handler)
```ts
export default {
  async fetch(request: Request, env: Env) { /* ...existing routes... */ },

  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext): Promise<void> {
    const { results } = await env.DB.prepare(
      `SELECT title, author FROM recipes ORDER BY created_at DESC LIMIT 5`
    ).all();

    console.log("Nightly digest:", JSON.stringify(results));
    // Real app: call an email API (e.g. Resend, Postmark) here.
  },
};
```

## Free Tier Reality Check
- Cron Triggers themselves are free; the work your `scheduled()` handler does still counts against the same Worker CPU-time-per-invocation limit (10ms free plan) — so keep scheduled jobs light or move heavy work into a Queue message instead.
- Cap on the number of cron triggers per Worker: currently a handful (check the limits doc for the exact current number before recording, it's small — usually enough for one project).

## CTA
"Time for the part everyone's been waiting for: Workers AI. Let's generate a recipe from a list of ingredients."
