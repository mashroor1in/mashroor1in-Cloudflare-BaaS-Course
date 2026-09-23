# Lesson 7 — "Cloudflare KV: Caching & Session Storage in Under 10 Minutes"

**Length target:** 8–10 min | **Type:** Hands-on

## Hook
- "D1 is great, but you don't want to hit a database on every single page load. Here's how to cache with KV."

## Talking points
- KV = a global, eventually-consistent key-value store, read-optimized, distributed to Cloudflare's edge.
- Perfect for: cached query results, session tokens, feature flags, config — not for things that need instant strong consistency or heavy write volume.
- Create the namespace, bind it, cache the "trending recipes" list with a TTL.
- Explain `expirationTtl` and why it matters for the 1,000 writes/day free cap — cache invalidation via TTL costs zero extra writes.
- Cache-aside pattern: check KV first, fall back to D1, write back to KV.

## Build steps
1. `npx wrangler kv namespace create PANTRYCHEF_CACHE`
2. Add binding to `wrangler.jsonc` with the returned id.
3. Update the `GET /recipes` route to read-through KV.

## Code — wrangler.jsonc (add)
```jsonc
{
  "kv_namespaces": [
    { "binding": "CACHE", "id": "<paste-from-wrangler-kv-namespace-create-output>" }
  ]
}
```

## Code — src/index.ts (replace GET /recipes handler)
```ts
export interface Env {
  DB: D1Database;
  CACHE: KVNamespace;
}

const TRENDING_KEY = "trending-recipes";
const TRENDING_TTL_SECONDS = 60; // cache for 1 minute

async function getTrending(env: Env) {
  const cached = await env.CACHE.get(TRENDING_KEY, "json");
  if (cached) return { data: cached, source: "kv-cache" };

  const { results } = await env.DB.prepare(
    `SELECT id, title, author, created_at FROM recipes ORDER BY created_at DESC LIMIT 50`
  ).all();

  await env.CACHE.put(TRENDING_KEY, JSON.stringify(results), {
    expirationTtl: TRENDING_TTL_SECONDS,
  });

  return { data: results, source: "d1" };
}

// inside fetch():
// if (pathname === "/recipes" && request.method === "GET") {
//   const { data, source } = await getTrending(env);
//   return Response.json({ recipes: data, source });
// }
```

## Free Tier Reality Check
- ~1GB storage, ~100,000 reads/day, ~1,000 writes/day.
- Reads are cheap and plentiful; writes are the scarce resource — this is why we use a TTL-based cache-aside pattern instead of writing to KV on every recipe creation.
- Show the `source: "kv-cache"` vs `source: "d1"` field flipping in the response as the TTL expires — makes the caching layer visible on camera.

## CTA
"Recipes need photos. Next: Cloudflare R2 — S3-compatible storage with zero egress fees."
