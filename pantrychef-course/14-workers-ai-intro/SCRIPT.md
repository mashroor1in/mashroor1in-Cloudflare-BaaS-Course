# Lesson 14 — "Cloudflare Workers AI: Run LLMs, Image & Embedding Models for Free (10K Neurons/Day)"

**Length target:** 12–15 min | **Type:** Hands-on, high-CTR topic — consider filming/publishing this earlier than its folder number

## Hook
- "Free GPU-powered AI inference, one binding away. Let's generate a full recipe from a list of ingredients."

## Talking points
- Workers AI runs open models (Llama, Mistral, embedding models, image models) on Cloudflare's GPUs — you call them like any other binding, no separate API key or provider account needed.
- "Neurons" are Cloudflare's normalized compute unit — different models cost wildly different numbers of Neurons per call; show the model catalog page and point out a couple of cheap vs expensive models.
- Build `POST /generate-recipe`: take a list of ingredients, prompt a text-generation model, return structured JSON.
- Show a live generation on screen with 3–4 different ingredient lists for variety.

## Build steps
1. Add the `ai` binding to `wrangler.jsonc` (no separate resource to create — it's built in).
2. Add the `/generate-recipe` route calling `env.AI.run(...)`.
3. Test with a few ingredient combos via `curl` or the upload form.

## Code — wrangler.jsonc (add)
```jsonc
{
  "ai": { "binding": "AI" }
}
```

## Code — src/index.ts (add route)
```ts
export interface Env {
  AI: Ai;
}

// inside fetch():
if (pathname === "/generate-recipe" && request.method === "POST") {
  const { ingredients } = await request.json<{ ingredients: string[] }>();

  const prompt = `You are a helpful chef. Using ONLY these ingredients (plus basic pantry staples like salt, oil, water): ${ingredients.join(", ")}.
Return a JSON object with keys: title (string), steps (array of strings). No extra commentary.`;

  const result = await env.AI.run("@cf/meta/llama-3.1-8b-instruct", {
    messages: [{ role: "user", content: prompt }],
  });

  return Response.json({ raw: result.response });
}
```

## Free Tier Reality Check
- 10,000 Neurons/day, resets daily at 00:00 UTC — this is an ongoing daily allowance, not a one-time signup credit.
- Framing for viewers: how far 10,000 Neurons goes depends entirely on the model — a small text model might support hundreds of generations/day, while a large image model burns the pool far faster. Show the live Neuron cost table for 2–3 models on screen.
- Overage cost if you exceed it: $0.011 per 1,000 Neurons — cheap, but worth mentioning so nobody's surprised.

## CTA
"Let's make this AI stateful — next we build 'Ask the Chef,' a chatbot that remembers the conversation using D1."
