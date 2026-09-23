# Lesson 9 — "File Uploads End-to-End: Pages Frontend → Worker → R2"

**Length target:** 10–12 min | **Type:** Hands-on integration

## Hook
- "Let's connect everything so far into one real feature: a form on the site that uploads a photo straight into R2."

## Talking points
- Walk through the full request path on a whiteboard/diagram: browser form → `fetch()` to the Worker API → Worker streams to R2 → Worker returns the public URL → frontend displays it.
- CORS: since Pages and the Worker are on different subdomains, you need to set CORS headers on the Worker — a very common beginner blocker, worth 2 full minutes on screen.
- Show the network tab in devtools during the upload so viewers can see the request/response.

## Build steps
1. Add an `OPTIONS` handler + CORS headers to the Worker (see lesson 5's config pattern for allowed origin).
2. Build `public/upload.html` with a simple form.
3. Test end-to-end: pick a file, upload, see the returned URL, load the image.

## Code — src/index.ts (add CORS)
```ts
const ALLOWED_ORIGIN = "https://pantrychef.pages.dev"; // swap for your real Pages domain

function withCors(response: Response): Response {
  const headers = new Headers(response.headers);
  headers.set("Access-Control-Allow-Origin", ALLOWED_ORIGIN);
  headers.set("Access-Control-Allow-Methods", "GET, POST, PUT, OPTIONS");
  headers.set("Access-Control-Allow-Headers", "Content-Type");
  return new Response(response.body, { status: response.status, headers });
}

// inside fetch(), before routing:
if (request.method === "OPTIONS") {
  return withCors(new Response(null, { status: 204 }));
}
// wrap every returned Response in withCors(...) before sending
```

## Code — public/upload.html
```html
<!doctype html>
<html lang="en">
<head><meta charset="UTF-8" /><title>Upload a Recipe Photo</title></head>
<body style="font-family: system-ui; max-width: 480px; margin: 60px auto;">
  <h2>Upload a recipe photo</h2>
  <input type="file" id="file" accept="image/*" />
  <button id="go">Upload</button>
  <p id="result"></p>
  <img id="preview" style="max-width:100%; margin-top:16px;" />

  <script>
    const API = "https://pantrychef-api.<subdomain>.workers.dev";
    document.getElementById("go").onclick = async () => {
      const file = document.getElementById("file").files[0];
      if (!file) return alert("Pick a file first");
      const key = crypto.randomUUID() + "-" + file.name;
      const res = await fetch(`${API}/photos/${key}`, {
        method: "PUT",
        headers: { "Content-Type": file.type },
        body: file,
      });
      const data = await res.json();
      document.getElementById("result").textContent = "Uploaded: " + data.key;
      document.getElementById("preview").src = `${API}/photos/${data.key}`;
    };
  </script>
</body>
</html>
```

## Free Tier Reality Check
- Same caps as lessons 2 and 8 apply (100K Worker requests/day, 10GB R2 storage) — the new thing to flag here is Worker CPU time when streaming larger files; keep uploads client-side-compressed if you expect big images.

## CTA
"Time for something more advanced: Durable Objects, for real-time state — we'll build a live Cook-Along room."
