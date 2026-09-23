# Lesson 2 — "Deploy Your First Cloudflare Worker in 5 Minutes"

**Length target:** 6–8 min | **Type:** Hands-on build

## Hook (0:00–0:15)
- "In the next 5 minutes you're going to write, deploy, and hit a live API endpoint running on Cloudflare's global network — for free, no credit card."

## Talking points
- What a Worker actually is: a JS/TS function that runs on Cloudflare's edge, close to the user, instead of one server in one region.
- `npm create cloudflare@latest` walkthrough — pick "Hello World Worker."
- Explain `wrangler.jsonc` (name, main, compatibility_date) — this file will grow across the series.
- `wrangler dev` for local testing, then `wrangler deploy` to go live.
- Hit the deployed URL live on screen, show the response.
- Show the Cloudflare dashboard: Workers > your worker > Metrics tab, to make "100,000 requests/day" concrete.

## Build steps
1. `npm create cloudflare@latest pantrychef-api -- --type=hello-world`
2. `cd pantrychef-api`
3. `npx wrangler dev` → test locally
4. `npx wrangler deploy` → get live `*.workers.dev` URL
5. `curl https://pantrychef-api.<subdomain>.workers.dev`

## Code — src/index.ts
```ts
export interface Env {}

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === "/") {
      return Response.json({
        message: "Welcome to the PantryChef API",
        status: "ok",
      });
    }

    if (url.pathname === "/health") {
      return new Response("ok", { status: 200 });
    }

    return new Response("Not found", { status: 404 });
  },
};
```

## Code — wrangler.jsonc
```jsonc
{
  "name": "pantrychef-api",
  "main": "src/index.ts",
  "compatibility_date": "2026-09-01",
  "observability": { "enabled": true }
}
```

## Free Tier Reality Check
- 100,000 requests/day, 10ms CPU time/request, 128MB memory, 50 subrequests/request.
- At 100K req/day that's ~69 requests/minute sustained, all day, every day — plenty for a side project or early-stage app.
- What breaks it: a request that loops or calls a slow external API without `waitUntil` can burn CPU time and throw a `1102` error. Show this on screen by adding a deliberate `while(true)` for 2 seconds — safe, fast demo of the limit.

## CTA
"Next video: turn this into a real website with Cloudflare Pages — free custom domain included."
