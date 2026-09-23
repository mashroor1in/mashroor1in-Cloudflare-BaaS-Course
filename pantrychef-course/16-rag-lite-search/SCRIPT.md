# Lesson 16 — "RAG on a Budget: Workers AI Embeddings + D1/KV as a Free Vectorize Alternative"

**Length target:** 15–18 min | **Type:** Hands-on, advanced/differentiated content

## Hook
- "Vectorize — Cloudflare's real vector database — isn't on the free plan. Here's how to build ingredient-based semantic search anyway, using only free products."

## Talking points
- What a vector/embedding actually is, in plain terms: a list of numbers that represents the *meaning* of text, so "similar meaning" texts have "close" vectors.
- The trick: generate embeddings with a free Workers AI embedding model, store the resulting vector as JSON in a D1 column (or KV), then compute cosine similarity **in the Worker** at query time instead of using a dedicated vector database.
- Be explicit about the trade-off on screen: this works great up to a few thousand recipes; past that, brute-force cosine similarity in a Worker gets slow and CPU-time-limited — that's the point where Vectorize (paid) becomes worth it. This honesty is the differentiator of this video.
- Live demo: search "something with eggs and cheese" and get semantically relevant recipes back, not just keyword matches.

## Build steps
1. Add an `embedding` column to `recipes` (store as JSON text).
2. On recipe creation, generate an embedding from the title+ingredients and store it.
3. Build `GET /search?q=...`: embed the query, pull all recipe embeddings, rank by cosine similarity, return the top N.

## Code — schema addition
```sql
ALTER TABLE recipes ADD COLUMN embedding TEXT;
```

## Code — src/index.ts (embedding helpers + route)
```ts
async function embed(env: Env, text: string): Promise<number[]> {
  const result = await env.AI.run("@cf/baai/bge-base-en-v1.5", { text: [text] });
  return result.data[0];
}

function cosineSimilarity(a: number[], b: number[]): number {
  let dot = 0, normA = 0, normB = 0;
  for (let i = 0; i < a.length; i++) {
    dot += a[i] * b[i];
    normA += a[i] * a[i];
    normB += b[i] * b[i];
  }
  return dot / (Math.sqrt(normA) * Math.sqrt(normB));
}

// On recipe creation (add to POST /recipes, after the INSERT):
const embedding = await embed(env, `${body.title} ${body.ingredients.join(", ")}`);
await env.DB.prepare(`UPDATE recipes SET embedding = ? WHERE id = ?`)
  .bind(JSON.stringify(embedding), recipeId).run();

// New route:
if (pathname === "/search" && request.method === "GET") {
  const q = url.searchParams.get("q") ?? "";
  const queryVector = await embed(env, q);

  const { results } = await env.DB.prepare(
    `SELECT id, title, embedding FROM recipes WHERE embedding IS NOT NULL LIMIT 500`
  ).all();

  const ranked = (results as any[])
    .map((r) => ({ id: r.id, title: r.title, score: cosineSimilarity(queryVector, JSON.parse(r.embedding)) }))
    .sort((a, b) => b.score - a.score)
    .slice(0, 10);

  return Response.json(ranked);
}
```

## Free Tier Reality Check
- Embedding calls draw from the same 10,000 Neurons/day pool as lesson 14 — each search and each recipe creation costs one embedding call.
- The `LIMIT 500` in the query is a deliberate free-tier guardrail: it caps how much data comes back from D1 and how much cosine-similarity math the Worker does per request, keeping it inside the 10ms CPU budget on the free plan. Show what happens (a `1102` CPU-limit error) if you remove the limit against a large dataset — great "here's the wall" demo moment.
- Explicitly state: this is a teaching pattern for small/medium datasets, not a production vector search replacement.

## CTA
"Let's add one more AI-adjacent feature: importing a recipe straight from a URL, using Browser Rendering."
