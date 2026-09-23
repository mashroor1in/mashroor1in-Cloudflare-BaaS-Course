# Lesson 17 — "Free Browser Automation: Cloudflare Browser Rendering for Scraping/Screenshots"

**Length target:** 10–12 min | **Type:** Hands-on

## Hook
- "Let's build 'paste a URL, get a recipe' — using a real headless browser, running for free inside a Worker."

## Talking points
- Browser Rendering gives you a real headless Chrome instance as a Worker binding — useful for scraping JS-rendered pages, taking screenshots, or generating PDFs, without running your own browser infrastructure.
- Use Puppeteer's API (Cloudflare's binding is Puppeteer-compatible) to open a page, extract text, and close the browser promptly — session limits make "closing fast" important.
- PantryChef use case: user pastes a recipe blog URL, the Worker renders the page, extracts the visible text, and feeds it to the Workers AI model from lesson 14 to structure it into title/ingredients/steps.

## Build steps
1. Enable Browser Rendering binding in `wrangler.jsonc`.
2. Add `POST /import-from-url`: launch browser, navigate, extract text, close browser, pass text to the AI model for structuring.
3. Demo on a real public recipe blog URL.

## Code — wrangler.jsonc (add)
```jsonc
{
  "browser": { "binding": "BROWSER" }
}
```

## Code — src/index.ts (add route)
```ts
import puppeteer from "@cloudflare/puppeteer";

export interface Env {
  BROWSER: Fetcher;
  AI: Ai;
}

if (pathname === "/import-from-url" && request.method === "POST") {
  const { url: recipeUrl } = await request.json<{ url: string }>();

  const browser = await puppeteer.launch(env.BROWSER);
  const page = await browser.newPage();
  await page.goto(recipeUrl, { waitUntil: "domcontentloaded" });
  const pageText = await page.evaluate(() => document.body.innerText);
  await browser.close(); // close promptly — concurrent session limits are tight on the free plan

  const prompt = `Extract a recipe from this page text. Return JSON with keys title, ingredients (array), steps (array). Text:\n\n${pageText.slice(0, 4000)}`;
  const result = await env.AI.run("@cf/meta/llama-3.1-8b-instruct", {
    messages: [{ role: "user", content: prompt }],
  });

  return Response.json({ raw: result.response });
}
```

## Free Tier Reality Check
- 10 minutes of browser usage per day, 3 concurrent browsers, 3 new browser instances per minute on the free plan.
- Practical tip to show on screen: always `browser.close()` as soon as you're done — sessions that sit idle eat into both the daily minutes and the concurrency cap.
- Framing: 10 minutes/day sounds small, but a single page render + extraction usually takes 1–3 seconds, so this comfortably covers dozens of imports a day for a side project.

## CTA
"Let's make all of this measurable — next, a free analytics dashboard with Analytics Engine."
