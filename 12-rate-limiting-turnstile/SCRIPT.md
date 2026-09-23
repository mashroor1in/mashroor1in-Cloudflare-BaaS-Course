# Lesson 12 — "Rate Limiting & Turnstile: Securing the Free Plan"

**Length target:** 8–10 min | **Type:** Hands-on

## Hook
- "A public submission form without protection is a bot magnet. Two free tools fix that: rate limiting and Turnstile."

## Talking points
- Turnstile = Cloudflare's free, privacy-friendly CAPTCHA alternative — no puzzles, mostly invisible to real users.
- Simple in-Worker rate limiting using KV as a counter (good enough for most side projects; mention the dedicated Rate Limiting API/WAF rules as the "when you need more" upgrade).
- Walk through adding the Turnstile widget to the upload/recipe-submission form and verifying the token server-side in the Worker before writing to D1.

## Build steps
1. Add Turnstile site in the Cloudflare dashboard (Turnstile tab), get site key + secret key.
2. `wrangler secret put TURNSTILE_SECRET`
3. Add the Turnstile widget script to the recipe submission form.
4. Verify the token server-side before accepting the `POST /recipes` request.
5. Add a simple KV-based rate limiter (e.g., 5 requests/minute per IP) in front of the same route.

## Code — src/index.ts (Turnstile verification + rate limit helper)
```ts
export interface Env {
  TURNSTILE_SECRET: string;
  CACHE: KVNamespace; // reused from lesson 7
}

async function verifyTurnstile(token: string, secret: string, ip: string): Promise<boolean> {
  const res = await fetch("https://challenges.cloudflare.com/turnstile/v0/siteverify", {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: new URLSearchParams({ secret, response: token, remoteip: ip }),
  });
  const data = await res.json<{ success: boolean }>();
  return data.success;
}

async function isRateLimited(env: Env, ip: string): Promise<boolean> {
  const key = `ratelimit:${ip}`;
  const count = Number((await env.CACHE.get(key)) ?? "0");
  if (count >= 5) return true;
  await env.CACHE.put(key, String(count + 1), { expirationTtl: 60 });
  return false;
}

// inside POST /recipes, before inserting into D1:
const ip = request.headers.get("CF-Connecting-IP") ?? "unknown";
if (await isRateLimited(env, ip)) {
  return new Response("Too many requests", { status: 429 });
}
const { turnstileToken, ...body } = await request.json<any>();
const human = await verifyTurnstile(turnstileToken, env.TURNSTILE_SECRET, ip);
if (!human) {
  return new Response("Verification failed", { status: 403 });
}
```

## Code — public snippet (add to the submission form's HTML)
```html
<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>
<div class="cf-turnstile" data-sitekey="YOUR_SITE_KEY"></div>
```

## Free Tier Reality Check
- Turnstile: free, unlimited on Cloudflare's plans — no separate quota to worry about.
- KV-based rate limiting rides on the same ~1,000 writes/day free cap from lesson 7 — flag that a genuinely high-traffic form needs the dedicated Rate Limiting product (paid) instead of the KV trick shown here.

## CTA
"Let's automate something on a schedule next: a nightly trending-recipes digest using Cron Triggers."
