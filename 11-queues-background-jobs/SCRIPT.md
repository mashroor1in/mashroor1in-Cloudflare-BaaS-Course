# Lesson 11 — "Cloudflare Queues: Background Jobs and Async Processing for Free"

**Length target:** 10–12 min | **Type:** Hands-on

## Hook
- "Nobody should wait 3 seconds for an upload response while we resize an image and send an email. Let's move that work to the background — for free."

## Talking points
- Producer/consumer model: one Worker *produces* messages onto a queue; a separate Worker (or the same one, different handler) *consumes* them asynchronously.
- Use case in PantryChef: when a recipe is created, enqueue a "new-recipe" job that (a) logs an analytics event and (b) would call an email service in a real app.
- Show the immediate response to the user vs. the delayed background processing in the logs — this contrast is the whole point, make it visible on screen.
- Mention batching: consumers can process multiple messages per invocation, which is efficient against the Workers request cap.

## Build steps
1. `npx wrangler queues create pantrychef-jobs`
2. Add producer binding + consumer config to `wrangler.jsonc`.
3. Enqueue a message from the `POST /recipes` handler.
4. Add a `queue()` handler in the same (or a separate) Worker to process it.

## Code — wrangler.jsonc (add)
```jsonc
{
  "queues": {
    "producers": [
      { "binding": "RECIPE_JOBS", "queue": "pantrychef-jobs" }
    ],
    "consumers": [
      { "queue": "pantrychef-jobs", "max_batch_size": 10, "max_batch_timeout": 5 }
    ]
  }
}
```

## Code — src/index.ts (enqueue on creation, add queue handler)
```ts
export interface Env {
  RECIPE_JOBS: Queue;
  // ...other bindings
}

// inside the POST /recipes handler, after inserting into D1:
await env.RECIPE_JOBS.send({ type: "new-recipe", recipeId, title: body.title });

export default {
  async fetch(request: Request, env: Env) { /* ...existing routes... */ },

  async queue(batch: MessageBatch, env: Env): Promise<void> {
    for (const message of batch.messages) {
      const job = message.body as { type: string; recipeId: string; title: string };
      console.log(`Processing job: ${job.type} for "${job.title}" (${job.recipeId})`);
      // Real app: generate a thumbnail, send a welcome email, notify followers, etc.
      message.ack();
    }
  },
};
```

## Free Tier Reality Check
- Queues were added to the Workers free plan in February 2026 — this is a genuinely new, easy-to-miss free feature worth calling out explicitly as "recently free."
- Framing for viewers: background jobs used to require a separate service (SQS, a cron worker, a job runner) — now it's one binding in the same repo.

## CTA
"Let's lock down the submission form next — rate limiting and a free CAPTCHA with Turnstile."
