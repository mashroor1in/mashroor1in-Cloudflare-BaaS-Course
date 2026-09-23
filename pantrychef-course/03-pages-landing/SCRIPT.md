# Lesson 3 — "Cloudflare Pages 101: Deploy a Static Site + Custom Domain (Free SSL)"

**Length target:** 8–10 min | **Type:** Hands-on build

## Hook
- "Free hosting, free SSL, free custom domain, unlimited bandwidth. This is the PantryChef landing page, live in under 10 minutes."

## Talking points
- Pages vs a plain Worker: Pages is built for static sites + Git-based deploys; it also runs "Functions" (small Workers) alongside static assets.
- Two deploy paths: connect a GitHub repo (auto-deploy on push) or `wrangler pages deploy` from the CLI. Show the Git-connected path since it sets up the CI/CD habit early.
- Live: connect repo in the dashboard, watch the first build run.
- Add a custom domain (even a free subdomain like `.pages.dev` counts) and show automatic SSL.
- Mention the 500 builds/month and 20,000-files-per-deployment limits now, so it's not a surprise in lesson 19 (CI/CD).

## Build steps
1. Create `public/index.html` (landing page for PantryChef).
2. `git init && git add . && git commit -m "PantryChef landing page"`
3. Push to GitHub, connect the repo in Cloudflare Pages dashboard, set build output directory to `public`.
4. Watch first deploy, visit the `*.pages.dev` URL.
5. (Optional) Add a custom domain under Pages > Custom domains.

## Code — public/index.html
```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>PantryChef — Cook Together, Powered by Free AI</title>
  <style>
    body { font-family: system-ui, sans-serif; max-width: 640px; margin: 80px auto; padding: 0 20px; color: #1a1a1a; }
    h1 { font-size: 2rem; }
    .tag { display:inline-block; background:#f6821f; color:white; padding:4px 10px; border-radius:6px; font-size:0.8rem; }
  </style>
</head>
<body>
  <span class="tag">Built on Cloudflare's Free Tier</span>
  <h1>PantryChef</h1>
  <p>Generate recipes from what's in your fridge, cook along live with friends, and search by ingredient — all running on $0/month infrastructure.</p>
  <p><em>Landing page deployed with Cloudflare Pages. API coming in the next lesson.</em></p>
</body>
</html>
```

## Free Tier Reality Check
- 500 builds/month, unlimited static requests and bandwidth, 20,000 files per deployment.
- Static asset requests are unlimited and free — you only pay attention to the Workers 100K/day cap once you add Pages *Functions* (covered in lesson 9).

## CTA
"Next: a quick decision-tree video — when do you reach for a Worker vs Pages? Then we start wiring up the real backend."
