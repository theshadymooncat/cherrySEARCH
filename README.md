# cherrySEARCH

A glassmorphic search frontend that aggregates results via **SearXNG** with an optional **AI-powered overview** sidebar. Pink accents, frosted glass, cursor-reactive effects.

## Architecture

```
                    ┌─────────────────────┐
                    │   GitHub Pages       │
                    │  cherrySEARCH UI     │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  CORS Proxy Worker   │
                    │  (Cloudflare Worker) │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  SearXNG Instance     │
                    │  (local or public)    │
                    └──────────────────────┘
```

You need **three things**:

1. **A SearXNG instance** — local or public
2. **A CORS proxy** — Cloudflare Worker (recommended) to handle browser CORS restrictions
3. **This HTML file** — hosted anywhere (GitHub Pages, Vercel, locally, etc.)

Optionally, an **OpenRouter API key** enables the AI overview panel.

---

## Setup

### 1. Get a SearXNG instance

**Option A — Public instance** (easiest)
Use any public SearXNG instance. Set it in the settings panel (gear icon → SearXNG Instance).

**Option B — Self-hosted** (recommended)
Run your own SearXNG. If it's on your local machine, you'll need a tunnel to expose it (see step 2b).

### 2a. Deploy the CORS proxy (Cloudflare Worker)

Most SearXNG instances don't send CORS headers, so browsers block requests. A Cloudflare Worker fixes this.

1. Go to [cloudflare.com](https://cloudflare.com) → Workers & Pages → Create Worker
2. Paste this code:

```js
export default {
  async fetch(request) {
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
          'Access-Control-Allow-Headers': '*',
          'Access-Control-Max-Age': '86400',
        }
      });
    }
    const url = new URL(request.url).searchParams.get('url');
    if (!url) return new Response('Missing url param', { status: 400, headers: { 'Access-Control-Allow-Origin': '*' } });
    const res = await fetch(url);
    const body = await res.text();
    return new Response(body, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Content-Type': res.headers.get('Content-Type') || 'text/plain'
      }
    });
  }
}
```

3. Click **Deploy** — you get a URL like `https://your-worker.workers.dev`

### 2b. (Optional) Expose local SearXNG with Cloudflare Tunnel

If your SearXNG is on `localhost`, you need a tunnel to make it publicly accessible so the worker can reach it.

1. Download [cloudflared](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/)
2. Run:
   ```bash
   cloudflared tunnel --url http://localhost:[your port number]
   ```
3. It prints a URL like `https://something.trycloudflare.com` — use that as your SearXNG instance URL.

### 3. Configure cherrySEARCH

Open the HTML file or your deployed site, click the **gear icon** (⚙️), and fill in:

- **SearXNG Instance** — your instance URL (e.g. `https://searx.linxx.net` or your tunnel URL)
- **Use CORS proxy** — toggle on
- **CORS proxy URL** — your worker URL with `?url=` suffix (e.g. `https://your-worker.workers.dev/?url=`)
- **AI Overview** — toggle on, paste your OpenRouter API key, and optionally set a model

Settings are saved to `localStorage` — they persist between sessions.

### 4. Deploy to GitHub Pages

1. Push the repo to GitHub
2. Go to repo → Settings → Pages
3. Source: **Deploy from a branch** → `main` → `/` (root)
4. The file must be named **`index.html`** for the root URL to work

Your site will be at `https://your-username.github.io/repo-name/`

---

## Features

- **3-column layout** — AI overview, search results, Wikipedia sidebar
- **Liquid glass UI** — frosted glass, pink accents, backdrop blur
- **Cursor-reactive** — grid reveal, glow, and 3D card tilt (desktop only)
- **Multiple search categories** — General, Images, News, Files, Videos, Music, Social, IT
- **Infinite scroll** — loads more results as you scroll
- **Falling petals** — subtle ambient animation for glass refraction effect
- **Dark/light theme** — follows system preference, toggleable
- **Fully responsive** — works on desktop and mobile

---

## Tech

- **Single-file HTML** — no build tools, no dependencies
- **SearXNG JSON API** — open-source metasearch engine
- **Cloudflare Workers** — free edge CORS proxy
- **OpenRouter** — AI model API for search overviews
- **Cloudflare Tunnel** — expose local services for free
