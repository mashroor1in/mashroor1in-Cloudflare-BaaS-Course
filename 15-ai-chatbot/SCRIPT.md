# Lesson 15 — "Build an AI Chatbot API With Workers AI + D1 for Chat History"

**Length target:** 12–15 min | **Type:** Hands-on

## Hook
- "'Ask the Chef' — a cooking assistant that remembers your conversation. This is Workers AI and D1 working together."

## Talking points
- The core pattern: every chat message is stored in D1, and on each new message you pull recent history back out and include it in the prompt — this *is* how "memory" works for these chat APIs.
- Session ID pattern: generate a UUID client-side, store it (e.g., in localStorage), pass it on every request so the Worker knows which conversation to load.
- Show the schema for `chat_messages` (session_id, role, content, created_at).
- Live demo: ask a follow-up question that only makes sense with memory of the earlier message ("What about a vegetarian version?").

## Build steps
1. Extend the schema with a `chat_messages` table.
2. Add `POST /chat` route: save the user message, load recent history, call Workers AI with the full message list, save the assistant reply, return it.

## Code — schema addition
```sql
CREATE TABLE IF NOT EXISTS chat_messages (
  id TEXT PRIMARY KEY,
  session_id TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('user','assistant')),
  content TEXT NOT NULL,
  created_at INTEGER NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_chat_session ON chat_messages(session_id, created_at);
```

## Code — src/index.ts (add route)
```ts
const SYSTEM_PROMPT = "You are a friendly, concise cooking assistant for the PantryChef app. Keep answers short and practical.";

if (pathname === "/chat" && request.method === "POST") {
  const { sessionId, message } = await request.json<{ sessionId: string; message: string }>();

  await env.DB.prepare(
    `INSERT INTO chat_messages (id, session_id, role, content, created_at) VALUES (?, ?, 'user', ?, ?)`
  ).bind(crypto.randomUUID(), sessionId, message, Date.now()).run();

  const { results: history } = await env.DB.prepare(
    `SELECT role, content FROM chat_messages WHERE session_id = ? ORDER BY created_at ASC LIMIT 20`
  ).bind(sessionId).all();

  const result = await env.AI.run("@cf/meta/llama-3.1-8b-instruct", {
    messages: [{ role: "system", content: SYSTEM_PROMPT }, ...(history as any[])],
  });

  const reply = result.response as string;

  await env.DB.prepare(
    `INSERT INTO chat_messages (id, session_id, role, content, created_at) VALUES (?, ?, 'assistant', ?, ?)`
  ).bind(crypto.randomUUID(), sessionId, reply, Date.now()).run();

  return Response.json({ reply });
}
```

## Free Tier Reality Check
- Combines two free ceilings at once: D1's daily row read/write caps (lesson 6) and Workers AI's 10,000 Neurons/day (lesson 14) — each chat turn costs one D1 write, one D1 read (history), and one Neuron-consuming AI call.
- Practical tip for viewers: cap chat history length (we used `LIMIT 20`) — it keeps prompts small (fewer Neurons per call) and keeps D1 reads bounded.

## CTA
"Next: search. Since Vectorize isn't free, we'll build ingredient-based recipe search using embeddings and plain D1 — I call this 'RAG-lite.'"
