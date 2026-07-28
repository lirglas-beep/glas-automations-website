# glasautomations.com

One-page static site. No build step, no framework, no package manager.

## Layout

```
public/           <- everything here is served at the site root
  index.html        the whole site: markup, one <style> block, one <script> block
  robots.txt
  sitemap.xml
  _headers          Cloudflare response headers (PDF noindex, caching)
  assets/           logo, favicon mark, three one-pager PDFs
index-old.html    previous version, kept for reference. NOT deployed.
wrangler.jsonc    Cloudflare Worker config
```

Only `public/` is uploaded. Anything outside it stays in the repo.

## Deploying

Cloudflare is connected to this repo. **Push to `main` and it deploys.**

```bash
git add -A
git commit -m "your message"
git push
```

Cloudflare builds and rolls out in about a minute. Watch it under
Workers &amp; Pages -> glasautomations -> Deployments.

Do not drag-and-drop uploads into the dashboard any more — a manual upload and a
Git deploy will fight over which version is live.

## Editing

Everything is in `public/index.html`. Open it in a browser directly to check
copy changes. To exercise the Cal.com embed and the forms you need a real
server, because the asset paths are absolute (`/assets/...`):

```bash
npx serve public
```

## Things that will bite you

- **`name` in `wrangler.jsonc` must stay `glasautomations`.** It points at the
  existing Worker, which owns `glasautomations.com` and `www.glasautomations.com`.
  Rename it and you get a fresh Worker with no domains attached.
- **Asset paths are absolute.** `/assets/pricing.pdf`, not `assets/pricing.pdf`.
  They will 404 when opening the file over `file://` — that is expected.
- **If you ever zip this by hand, use forward slashes.** Windows tools write
  `assets\file.pdf` into the archive, Cloudflare stores that key literally, and
  every asset 404s while the HTML still loads fine.
- **`N8N_WEBHOOK_URL` at the top of the script block** must hold the real n8n
  production webhook URL. While it is the placeholder, both forms fall back to
  showing the mailto/text link instead of submitting.

## Where the forms go

Both forms POST JSON to the n8n workflow **Glas Automations - Website Contact Form**
(`xXGCdMnSVV84TtBe`), which normalizes the payload and emails it to
lir@glasautomations.com with Reply-To set to the sender when they left an email.

Submissions are not written to any spreadsheet or database — the email is the
only durable record.
