# Lesson 6 — "Add a Database for Free: Cloudflare D1 Crash Course"

**Length target:** 12–15 min | **Type:** Hands-on core build

## Hook
- "A real SQL database, replicated at the edge, 5GB free. Let's give PantryChef somewhere to store recipes."

## Talking points
- D1 is SQLite under the hood, but managed and replicated by Cloudflare — no server to provision.
- Create a database, bind it to the Worker, run migrations from the CLI.
- Show `wrangler d1 execute --local` vs without `--local` (local file vs remote database) — a very common beginner trip-up.
- Write the schema for `recipes` (id, title, ingredients, steps, author, created_at).
- Build CRUD routes: `POST /recipes`, `GET /recipes`, `GET /recipes/:id`.
- Show a query in the dashboard's D1 console as a sanity check.

## Build steps
1. `npx wrangler d1 create pantrychef-db`
2. Add the `d1_databases` binding to `wrangler.jsonc` (copy database_id from step 1's output).
3. `npx wrangler d1 execute pantrychef-db --local --file=schema/schema.sql` (local dev)
4. `npx wrangler d1 execute pantrychef-db --remote --file=schema/schema.sql` (deploy schema to production)
5. Wire up `src/index.ts` routes, test locally, then deploy.

## Code — schema/schema.sql
```sql
CREATE TABLE IF NOT EXISTS recipes (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  ingredients TEXT NOT NULL,   -- JSON array as text
  steps TEXT NOT NULL,         -- JSON array as text
  author TEXT,
  created_at INTEGER NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_recipes_created ON recipes(created_at DESC);
```

## Code — wrangler.jsonc (add)
```jsonc
{
  "d1_databases": [
    {
      "binding": "DB",
      "database_name": "pantrychef-db",
      "database_id": "<paste-from-wrangler-d1-create-output>"
    }
  ]
}
```

## Code — src/index.ts
```ts
export interface Env {
  DB: D1Database;
}

function id() {
  return crypto.randomUUID();
}

export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);
    const { pathname } = url;

    if (pathname === "/recipes" && request.method === "POST") {
      const body = await request.json<{ title: string; ingredients: string[]; steps: string[]; author?: string }>();
      const recipeId = id();
      await env.DB.prepare(
        `INSERT INTO recipes (id, title, ingredients, steps, author, created_at) VALUES (?, ?, ?, ?, ?, ?)`
      )
        .bind(recipeId, body.title, JSON.stringify(body.ingredients), JSON.stringify(body.steps), body.author ?? "anonymous", Date.now())
        .run();
      return Response.json({ id: recipeId }, { status: 201 });
    }

    if (pathname === "/recipes" && request.method === "GET") {
      const { results } = await env.DB.prepare(
        `SELECT id, title, author, created_at FROM recipes ORDER BY created_at DESC LIMIT 50`
      ).all();
      return Response.json(results);
    }

    const match = pathname.match(/^\/recipes\/([\w-]+)$/);
    if (match && request.method === "GET") {
      const row = await env.DB.prepare(`SELECT * FROM recipes WHERE id = ?`).bind(match[1]).first();
      if (!row) return new Response("Not found", { status: 404 });
      return Response.json({
        ...row,
        ingredients: JSON.parse(row.ingredients as string),
        steps: JSON.parse(row.steps as string),
      });
    }

    return new Response("Not found", { status: 404 });
  },
};
```

## Free Tier Reality Check
- 5GB storage on the free plan.
- Daily row read/write limits are now **enforced** (as of Sept 1, 2026) — exceeding them makes queries fail with an error until midnight UTC reset, rather than silently degrading. Show the actual D1 error in the dashboard logs if you can trigger it (e.g., a tight loop of inserts).
- Practical tip to show on screen: add an index on any column you filter/sort by (we did this for `created_at`) — it's the single biggest lever for staying under the row-read cap.

## CTA
"Next: caching. We'll use KV to serve trending recipes instantly without hitting D1 on every request."
