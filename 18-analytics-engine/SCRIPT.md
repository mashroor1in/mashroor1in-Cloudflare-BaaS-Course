# Lesson 18 — "Analytics Engine: Free Observability & Custom Metrics Dashboard"

**Length target:** 10–12 min | **Type:** Hands-on

## Hook
- "We've built a lot — let's see it working. Free, unlimited-cardinality analytics, no third-party tool needed."

## Talking points
- Analytics Engine lets you write custom data points (with arbitrary labels/dimensions) from inside a Worker, then query them back with SQL via the API — think "your own lightweight Mixpanel," free and built in.
- "Unlimited cardinality" means you can label events however you want (by recipe ID, by user, by route) without hitting per-dimension pricing like most analytics tools charge.
- Instrument the app: log an event on recipe views, recipe creations, AI generations, and chat messages.
- Query it back and render a tiny dashboard on the Pages frontend (a simple table or bar chart is enough for the demo).

## Build steps
1. Add an Analytics Engine binding in `wrangler.jsonc`.
2. Call `env.ANALYTICS.writeDataPoint(...)` at each key event across the app (recipe view, recipe create, AI generate, chat message).
3. Query recent data via the GraphQL Analytics API or SQL API from a small `/stats` route, return it as JSON for the dashboard.

## Code — wrangler.jsonc (add)
```jsonc
{
  "analytics_engine_datasets": [
    { "binding": "ANALYTICS", "dataset": "pantrychef_events" }
  ]
}
```

## Code — src/index.ts (log events, e.g. inside GET /recipes/:id)
```ts
export interface Env {
  ANALYTICS: AnalyticsEngineDataset;
}

env.ANALYTICS.writeDataPoint({
  blobs: ["recipe_view", match[1]],   // event type, recipe id
  doubles: [1],                        // count
  indexes: ["recipe_view"],            // primary index for fast filtering
});
```

## Free Tier Reality Check
- Analytics Engine is included on the free plan with unlimited cardinality for labels — the practical limit to mention is write volume per Worker invocation and query rate limits on the read API, which are generous for a side project's traffic.
- Good framing line for the video: "This is the free replacement for the analytics SaaS you were about to pay $29/month for."

## CTA
"One more thing before this is a real product: automatic deploys. Let's wire up CI/CD with GitHub Actions."
