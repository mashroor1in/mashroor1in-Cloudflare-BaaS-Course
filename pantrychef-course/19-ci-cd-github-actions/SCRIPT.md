# Lesson 19 — "CI/CD for Cloudflare: GitHub Actions + Wrangler Deploy Pipeline"

**Length target:** 10–12 min | **Type:** Hands-on, production polish

## Hook
- "Let's stop deploying by hand. Every push to main should test, then deploy, automatically — still free."

## Talking points
- GitHub Actions has a generous free tier for public repos, pairs naturally with Cloudflare's free tier for a genuinely $0 pipeline.
- Store a Cloudflare API token as a GitHub Actions secret (`CLOUDFLARE_API_TOKEN`), scoped to Workers/Pages edit only — show how to create a scoped token, not the global key, as a security best practice.
- Use `cloudflare/wrangler-action` to run `wrangler deploy` on push to `main`.
- Add a staging branch/environment (ties back to lesson 5's `env.staging` block) so PRs can deploy to staging before merging.

## Build steps
1. Create a scoped API token in the Cloudflare dashboard (Workers Scripts: Edit).
2. Add it as a repo secret: `CLOUDFLARE_API_TOKEN`.
3. Add the workflow file below.
4. Push a small change, watch the Action run in the GitHub Actions tab, confirm the live Worker updated.

## Code — .github/workflows/deploy.yml
```yaml
name: Deploy PantryChef API

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci

      - name: Deploy to staging (PRs)
        if: github.event_name == 'pull_request'
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          command: deploy --env staging

      - name: Deploy to production (main)
        if: github.ref == 'refs/heads/main'
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          command: deploy
```

## Free Tier Reality Check
- No new Cloudflare limit here — the thing to flag is that *every deploy still counts as one Worker deployment* and Pages builds still count toward the 500 builds/month cap (lesson 3), so a very chatty CI setup (deploying on every single commit to every branch) can burn that budget faster than expected.

## CTA
"Last video: let's be honest about money. Here's exactly when this stops being free, and what it costs to scale up."
