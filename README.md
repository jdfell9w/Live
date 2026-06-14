# Earth Cam WebXR V5 - Cloudflare Worker Refresh

This version keeps the WebXR scene on static hosting, but moves webcam fetching to a Cloudflare Worker.

## What changed from V4

- WebXR panels can now load camera stills through a Cloudflare Worker URL.
- The Worker adds CORS headers so the images can be used as WebGL/WebXR textures.
- The Worker caches each camera for 5 minutes.
- A Cron Trigger pre-warms the cache every 5 minutes.
- The WebXR page refreshes visible panel textures every 60 seconds with a cache-buster.
- If the Worker URL is not configured, the page falls back to local placeholder images in `/cams/`.

## Folder layout

```text
index.html
assets/earth_continent_outline_4096x2048.png
cams/*.jpg                  local placeholder fallback images
cloudflare-worker/          deploy this to Cloudflare Workers
```

## Deploy the Worker

```bash
cd cloudflare-worker
npm install
npx wrangler login
npx wrangler deploy
```

Wrangler will print a workers.dev URL like:

```text
https://earth-cam-webxr-proxy.YOUR_SUBDOMAIN.workers.dev
```

Test these URLs in a normal browser:

```text
https://earth-cam-webxr-proxy.YOUR_SUBDOMAIN.workers.dev/health
https://earth-cam-webxr-proxy.YOUR_SUBDOMAIN.workers.dev/cam/chicago.jpg
https://earth-cam-webxr-proxy.YOUR_SUBDOMAIN.workers.dev/cam/bardolino.jpg
https://earth-cam-webxr-proxy.YOUR_SUBDOMAIN.workers.dev/cam/shimizu.jpg
```

## Connect the WebXR page to the Worker

Option A: edit `index.html` and replace this line:

```js
const WORKER_BASE_DEFAULT = "https://REPLACE_WITH_YOUR_WORKER.workers.dev";
```

with your deployed Worker URL.

Option B: leave the file alone and open it with a query parameter:

```text
index.html?worker=https%3A%2F%2Fearth-cam-webxr-proxy.YOUR_SUBDOMAIN.workers.dev
```

## Local test

```bash
cd earth_cam_webxr_v5
python3 -m http.server 8000
```

Open:

```text
http://localhost:8000/index.html?worker=https%3A%2F%2Fearth-cam-webxr-proxy.YOUR_SUBDOMAIN.workers.dev
```

## GitHub Pages test

Upload the contents of this folder to GitHub Pages, then open:

```text
https://YOUR_USER.github.io/YOUR_REPO/index.html?worker=https%3A%2F%2Fearth-cam-webxr-proxy.YOUR_SUBDOMAIN.workers.dev
```

## Important limitation

The Worker can check/refetch every 5 minutes. It cannot force the original webcam owner to publish a new image every 5 minutes. Some source cameras only update every 10, 15, or 30 minutes, so some panels may occasionally show the same image across multiple refresh cycles.
