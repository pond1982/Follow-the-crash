# Follow the Crashes
### Bangkok road safety risk clusters · จุดเสี่ยงอุบัติเหตุบนถนนในกรุงเทพฯ

One hundred of the city's deadliest road intersections and stretches, drawn from municipal risk data, rendered as one interactive map. Each circle marks a cluster where people died or were hurt — sized by casualty count, colored by severity.

This is the first in a short series of open-data stories about Thailand. The same template will carry *Follow the Baht* (government procurement), *Election 2026* (House results by constituency), and *The PM2.5 Calendar* (daily air quality). One story, one page, one file.

## Running locally

The page needs to be served over HTTP — browsers block `fetch()` from `file://` URLs, so the district polygons won't load if you just double-click the file. From the folder containing `follow-the-crashes.html`:

```bash
python3 -m http.server 8000
# or
npx http-server -p 8000
# or
php -S localhost:8000
```

Then open <http://localhost:8000/follow-the-crashes.html>.

First paint shows the masthead, stat strip, and crash clusters instantly from bundled sample data. Bangkok's 50 district boundaries fetch asynchronously from GitHub raw and paint in after a few hundred milliseconds, then cache in `localStorage` for subsequent loads.

## Deploying

Any static host. Drop the single HTML file into Cloudflare Pages, Netlify, Vercel, GitHub Pages, an S3 bucket, or `scp` it to your own server. No build step, no dependencies, no server-side code.

## Data sources

| | |
|---|---|
| Crash clusters | [BMA City Data — 100 Risk Map](https://data.bangkok.go.th/dataset/100-risk-map) |
| Underlying accident data | ThaiRSC / iTIC |
| District polygons | [`pcrete/gsvloader-demo`](https://github.com/pcrete/gsvloader-demo) (GeoJSON, fetched at runtime) |
| Typography | [IBM Plex](https://www.ibm.com/plex/) — Sans, Sans Thai, Serif, Mono (SIL OFL) |

A sample dataset is bundled in the HTML so the page renders fully without any network connection — useful for visual QA and for shipping the file as a portable artifact.

## Going live with real BMA data

In the `<script>` block near the top of the file, find `LIVE_CONFIG`:

```javascript
const LIVE_CONFIG = {
  enabled:    false,
  endpoint:   "https://data.bangkok.go.th/th/api/3/action/datastore_search",
  resourceId: "PUT_RESOURCE_UUID_HERE",
  limit:      1000,
  fieldMap: { /* … */ }
};
```

1. Find a road-safety resource on [data.bangkok.go.th](https://data.bangkok.go.th). Open the resource page; its UUID is the last segment of the URL.
2. Paste the UUID into `resourceId`, set `enabled: true`.
3. Reload. The badge in the masthead transitions to `LIVE · N RECORDS` on success, or `LIVE FETCH FAILED` on error — in which case the page quietly falls back to sample data.

### On CORS

BMA's CKAN endpoint doesn't always send permissive CORS headers, so the browser may block the cross-origin fetch from your domain. Three ways around it, worst to best:

- **Public CORS proxy** (`corsproxy.io`, `allorigins`) — fine for a portfolio demo, not production.
- **Pre-scrape and commit** the JSON to your repo, served as a static file. Loses freshness, gains bulletproof delivery.
- **Proxy through your own domain** — Cloudflare Worker, Vercel API route, Next.js `/api/bma/[resource]`. Return the JSON with `Access-Control-Allow-Origin: *`. This is what you'd want to build anyway for the sibling stories.

## Architecture

A single HTML file, no framework, no build. Everything is plain HTML + inline CSS + vanilla JS.

- **SVG map, not a tile map.** No Leaflet, no MapLibre. District polygons and crash clusters are SVG nodes projected from lat/lng through a linear transform scoped to Bangkok's bounding box. Scales fluidly, weighs nothing.
- **Progressive enhancement.** District boundaries load async from GitHub raw and cache in `localStorage` under `bkk-districts-geojson-v1` — bump the version suffix to invalidate. If the fetch fails, the map degrades gracefully to centroid dots so the page is never blank.
- **Dark only, single token set.** `color-scheme: dark` is declared at `:root`. Every color passes WCAG AA contrast against its intended surface.
- **Fonts** load from Google Fonts for convenience. For production, self-host the Thai + Latin subsets at weights 400 / 500 — roughly 80KB total versus ~250KB from the CDN waterfall.

See [`STYLE_GUIDE.md`](./STYLE_GUIDE.md) for the full design system — color tokens, type scale, spacing, component patterns. It's the reference doc for building the sibling stories on the same template.

## Roadmap

- [x] **№ 01 · Follow the Crashes** — Bangkok road safety
- [ ] **№ 02 · Follow the Baht** — Thai government procurement contracts
- [ ] **№ 03 · Election 2026** — House of Representatives results by constituency
- [ ] **№ 04 · The PM2.5 Calendar** — daily air quality by district

Each story reuses the same template. What changes per issue: the severity palette, the hero stat's semantic meaning, the SVG content inside the map frame, and the colophon number.

## Credits

Built in Bangkok. Data from BMA City Data, ThaiRSC, and iTIC under Thailand's open government data license. District geography from [`pcrete/gsvloader-demo`](https://github.com/pcrete/gsvloader-demo). Typography is IBM Plex under SIL OFL. Template direction borrows quietly from Rest of World, Bloomberg Graphics, The Pudding, FT visual essays, and 101.world.

## License

Code: MIT. Data and geography retain their original licenses as linked above.
