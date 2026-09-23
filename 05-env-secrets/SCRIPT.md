# Lesson 5 — "Environment Variables, Secrets & Wrangler Config Done Right"

**Length target:** 6–8 min | **Type:** Hands-on

## Hook
- "Before we add a database or AI, let's get config right — this is where beginners leak API keys into GitHub."

## Talking points
- Difference between plain vars (`vars` in wrangler.jsonc, visible in the dashboard) and secrets (`wrangler secret put`, encrypted, never shown again).
- Local dev secrets live in `.dev.vars` (must be gitignored).
- Show `wrangler secret put OPENAI_KEY` (or any placeholder) live, then read it in code via `env.OPENAI_KEY`.
- Multiple environments (staging vs production) using `[env.staging]` / `[env.production]` blocks — set this up now so lesson 19 (CI/CD) is painless.
- `.gitignore` checklist on screen: `.dev.vars`, `.wrangler/`, `node_modules`.

## Build steps
1. `echo "APP_ENV=local" >> .dev.vars`
2. `npx wrangler secret put DEMO_SECRET` (type a throwaway value on screen)
3. Read both in a Worker route and return them (never do this in a real app — demo only, say so on screen).

## Code — wrangler.jsonc
```jsonc
{
  "name": "pantrychef-api",
  "main": "src/index.ts",
  "compatibility_date": "2026-09-01",
  "vars": {
    "APP_ENV": "production"
  },
  "env": {
    "staging": {
      "vars": { "APP_ENV": "staging" }
    }
  }
}
```

## Code — src/index.ts (add route)
```ts
export interface Env {
  APP_ENV: string;
  DEMO_SECRET: string;
}

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    if (url.pathname === "/config-check") {
      return Response.json({
        env: env.APP_ENV,
        secretIsSet: Boolean(env.DEMO_SECRET), // never return the actual secret value
      });
    }
    return new Response("Not found", { status: 404 });
  },
};
```

## Code — .gitignore
```
.dev.vars
.wrangler/
node_modules/
.env
```

## Free Tier Reality Check
- Free plan: 64 environment variables per Worker (Paid plan: 128).
- No cost implication here — this is a "save yourself from a leaked-key horror story" lesson.

## CTA
"Now that config is solid, let's add a real database — D1, Cloudflare's free serverless SQL database."
