# Lesson 10 — "Durable Objects Explained: Real-Time State on the Edge (Build a Live Cook-Along Room)"

**Length target:** 15–18 min | **Type:** Hands-on, advanced

## Hook
- "Workers are stateless by design — every request can hit a different edge location. Durable Objects fix that. Let's build a live room where people cook the same recipe together in real time."

## Talking points
- The core problem Durable Objects solve: you need *one* consistent place for shared state (a chat room, a counter, a game session) even though your Worker itself is stateless and globally distributed.
- Each Durable Object instance is a single-threaded, strongly-consistent mini-server with its own storage, addressed by a unique ID (e.g., recipe ID → one Cook-Along room per recipe).
- WebSockets: the Worker upgrades the connection and hands it to the Durable Object, which broadcasts messages to everyone connected to that same room.
- Live demo: open two browser tabs, join the same recipe room, show messages/step-checkmarks syncing instantly between them.

## Build steps
1. Define the `CookAlongRoom` Durable Object class.
2. Bind it in `wrangler.jsonc` with a migration.
3. Route `/cookalong/:recipeId` in the main Worker to `env.COOKALONG.idFromName(recipeId)`.
4. Build a minimal HTML page with a WebSocket client to demo it live.

## Code — wrangler.jsonc (add)
```jsonc
{
  "durable_objects": {
    "bindings": [
      { "name": "COOKALONG", "class_name": "CookAlongRoom" }
    ]
  },
  "migrations": [
    { "tag": "v1", "new_sqlite_classes": ["CookAlongRoom"] }
  ]
}
```

## Code — src/cookalong.ts
```ts
export class CookAlongRoom {
  state: DurableObjectState;
  sessions: Set<WebSocket> = new Set();

  constructor(state: DurableObjectState) {
    this.state = state;
  }

  async fetch(request: Request): Promise<Response> {
    if (request.headers.get("Upgrade") !== "websocket") {
      return new Response("Expected WebSocket", { status: 426 });
    }

    const pair = new WebSocketPair();
    const [client, server] = Object.values(pair);
    server.accept();
    this.sessions.add(server);

    server.addEventListener("message", (event) => {
      // Broadcast to everyone else in the same room
      for (const session of this.sessions) {
        if (session !== server && session.readyState === WebSocket.OPEN) {
          session.send(event.data as string);
        }
      }
    });

    server.addEventListener("close", () => this.sessions.delete(server));

    return new Response(null, { status: 101, webSocket: client });
  }
}
```

## Code — src/index.ts (add route)
```ts
export interface Env {
  COOKALONG: DurableObjectNamespace;
}

// inside fetch():
const roomMatch = pathname.match(/^\/cookalong\/([\w-]+)$/);
if (roomMatch) {
  const id = env.COOKALONG.idFromName(roomMatch[1]);
  const stub = env.COOKALONG.get(id);
  return stub.fetch(request);
}
```

## Free Tier Reality Check
- ~400,000 GB-seconds and ~1,000,000 requests/month on the free plan.
- Practical framing: a Durable Object only "costs" while active with connections — an idle cooking room with nobody in it costs nothing, which is why this pattern is cheap even at free-tier limits.
- What breaks it: thousands of *simultaneously active* rooms with long-lived connections; fine for a demo/side project, worth flagging as a scaling wall for a real product.

## CTA
"Next up: background jobs with Queues — we'll generate a thumbnail every time a new recipe photo comes in, without blocking the upload response."
