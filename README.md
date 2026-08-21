# Adbrew Dashboards

Static dashboards built from HubSpot data. One folder per dashboard, no build step,
no dependencies — each `index.html` is fully self-contained.

| Path | Dashboard | Source segment |
|---|---|---|
| `/` | Landing page | — |
| `/accelerate-survey-leads/` | Amazon Accelerate 2026 — Survey Leads | `Event Name = Amazon Accelerate 2026 Survey Respondent` AND `Lead status is none of Converted` |

## ⚠️ These pages contain personal data

They include names, work emails, job titles and employers of individual contacts, some of
whom are EU data subjects. **The Vercel deployment is public — there is no access control.**
`robots.txt` and `X-Robots-Tag: noindex` discourage search engines but do not restrict access.
Anyone with the URL can read the full contact list. Treat the URL as the secret.

To add a password later, see "Adding a password" below — it is a nine-line file.

## Deploy to Vercel

1. Push this repo to GitHub (already done if you're reading this there).
2. Vercel → **Add New… → Project → Import** `riteeshreddy-adbrew/Dashboards`.
3. Framework preset **Other**. Build command: leave empty. Output directory: leave empty.
4. Deploy. Every push to `main` redeploys automatically.

## Adding a new dashboard

```
mkdir amazon
# put the built page at amazon/index.html
```

Then add a card to `index.html` pointing at `/amazon/`. That's it — Vercel serves any
folder containing an `index.html` at its own path.

## Refreshing

Each dashboard is a point-in-time snapshot. To refresh, replace the `index.html` and push:

```bash
git add . && git commit -m "refresh $(date -u +%FT%TZ)" && git push
```

The scheduled Claude task rebuilds the Accelerate page every 2 hours on weekdays
(9am–7pm IST). Give it a Vercel deploy hook to make the hosted copy refresh too.

## Adding a password (free, Hobby plan)

Create `middleware.js` at the repo root:

```js
export const config = { matcher: '/((?!_next|favicon|robots).*)' };

export default function middleware(req) {
  const expected = 'Basic ' + btoa('team:' + process.env.DASH_PASSWORD);
  if (req.headers.get('authorization') === expected) return;
  return new Response('Authentication required', {
    status: 401,
    headers: { 'WWW-Authenticate': 'Basic realm="Adbrew dashboards"' },
  });
}
```

Set `DASH_PASSWORD` in Vercel → Settings → Environment Variables, then redeploy.
