# Lesson 8 — "R2 Storage: S3-Compatible Object Storage With Zero Egress Fees"

**Length target:** 8–10 min | **Type:** Hands-on

## Hook
- "This is the one Cloudflare product that genuinely undercuts AWS on a core cost — zero egress fees. Let's store recipe photos."

## Talking points
- R2 = S3-compatible object storage; same API shape (buckets, keys, `PUT`/`GET`), but Cloudflare doesn't charge you to serve the files out — that's the headline feature.
- Create a bucket, bind it to the Worker.
- Two upload patterns: direct-to-Worker upload (simple, subject to Worker CPU/subrequest limits) vs presigned URLs (better for large files) — build the simple one here, mention presigned URLs as the "when you outgrow this" upgrade.
- Serve images back out through a Worker route.

## Build steps
1. `npx wrangler r2 bucket create pantrychef-photos`
2. Add binding to `wrangler.jsonc`.
3. Add `PUT /photos/:key` and `GET /photos/:key` routes.

## Code — wrangler.jsonc (add)
```jsonc
{
  "r2_buckets": [
    { "binding": "PHOTOS", "bucket_name": "pantrychef-photos" }
  ]
}
```

## Code — src/index.ts (add routes)
```ts
export interface Env {
  PHOTOS: R2Bucket;
}

// inside fetch():
const photoMatch = pathname.match(/^\/photos\/([\w.-]+)$/);

if (photoMatch && request.method === "PUT") {
  const key = photoMatch[1];
  await env.PHOTOS.put(key, request.body, {
    httpMetadata: { contentType: request.headers.get("content-type") ?? "application/octet-stream" },
  });
  return Response.json({ key }, { status: 201 });
}

if (photoMatch && request.method === "GET") {
  const object = await env.PHOTOS.get(photoMatch[1]);
  if (!object) return new Response("Not found", { status: 404 });
  const headers = new Headers();
  object.writeHttpMetadata(headers);
  headers.set("etag", object.httpEtag);
  headers.set("cache-control", "public, max-age=31536000, immutable");
  return new Response(object.body, { headers });
}
```

## Demo
```bash
curl -X PUT --data-binary @sample-recipe.jpg \
  -H "Content-Type: image/jpeg" \
  https://pantrychef-api.<subdomain>.workers.dev/photos/sample-recipe.jpg

curl https://pantrychef-api.<subdomain>.workers.dev/photos/sample-recipe.jpg -o downloaded.jpg
```

## Free Tier Reality Check
- 10GB storage, generous free operations, and — the standout number — **zero egress fees**, meaning serving files out to users costs nothing even at scale, unlike S3.
- Practical framing for viewers: this is often the single biggest reason indie devs migrate file storage to Cloudflare even if the rest of their stack stays elsewhere.

## CTA
"Now let's connect the dots: a real upload form on the Pages frontend, through the Worker, into R2."
