# HANDOFF — Climate Haven Radars

A single-file interactive web tool: two editable "star" (radar) charts comparing candidate cities for a
family move. Built for Cameron; shared with family by link. This doc lets a fresh session (e.g. a cloud
session driven from Cameron's phone) pick up and make changes without re-deriving anything.

## Where it lives
- **Live site (share this):** https://cfredrickson79-cmyk.github.io/climate-haven-radars/
- **Repo:** https://github.com/cfredrickson79-cmyk/climate-haven-radars  (public, GitHub Pages from `main` / root)
- **Deployed file:** `index.html` at repo root — GitHub Pages serves it automatically on push.
- **Raw (for pulling current bytes):** https://raw.githubusercontent.com/cfredrickson79-cmyk/climate-haven-radars/main/index.html
- **GitHub account (Composio):** `cfredrickson79-cmyk` (connection alias `github_meith-scride`).

## What it does (current features)
- Two tabs: **Comfort & Experience** and **Financials**. Each has an editable grades table + a live SVG radar.
- Grades are **locked by default** (read-only) so viewers can't nudge values by accident. An **Unlock editing**
  button turns on add/remove city, add/remove parameter, rename, and value editing.
- **Click a numbered vertex** on the chart to pop that axis's full label; click again to hide.
- **Click a city name's ↗ link** to open an overview (Wikipedia; falls back to a Google search for cities with no `url`).
- Cell shading: value **≥ 8 = pastel green** (`#d9f2df`), **≤ 3 = pastel red** (`#fbe0e0`). Same shading on the Blended row.
- **Blended · radar area** row: each city scored by its radar polygon area, remapped to 0–10 via
  `10 * sqrt( Σ(v[i]·v[i+1]) / (100·N) )`. Rewards well-rounded profiles; depends on axis order.
- **Download PNG** (per active tab), **Export/Import JSON**, **Reset this radar**.
- Per-viewer persistence via `localStorage` key `climateHavenRadars_v3`.

## Architecture (all in `index.html`)
- Vanilla HTML/CSS/JS, no external libraries, no build step. Fully self-contained.
- `defaults()` returns the seed data object `{ comfort:{...}, financials:{...} }`.
  - Each dataset: `label`, `note`, `params:[axisName,...]`, `cities:[{name, url, color, values:[...]}]`.
  - `values` array length MUST equal `params` length for every city (6 axes currently in both datasets).
- Rendering: `renderTabs()`, `renderTable()`, `renderChart()` (SVG), all driven by `DATA[active]`.
- State: `DATA`, `active` ("comfort"|"financials"), `locked` (bool, default `true`), `openLabels` (Set per tab).
- Radar geometry: axis 0 at top (12 o'clock), clockwise; value 0 = center, 10 = outer ring; viewBox `0 0 520 470`.

### IMPORTANT gotchas
- **Bump `STORE_KEY`** (e.g. `_v3` → `_v4`) whenever you change the seed `defaults()` data. Otherwise returning
  visitors keep seeing the OLD data from their browser cache instead of your new defaults. `load()` also clears
  the previous keys.
- Keep `params` and every city's `values` the same length, or the chart/table desync.
- External links (Wikipedia/Google) are fine here (plain GitHub Pages, no CSP). Do not add remote scripts/styles/images.

## How to make a change (the reliable pipeline)
This project deploys through the **Composio GitHub API** — no browser needed. It transfers only a small diff and
gates on SHA-256 so a bad paste can never reach GitHub.

1. Pull current bytes from the raw URL; edit `index.html` locally.
2. If you changed `defaults()` data, bump `STORE_KEY`.
3. Verify locally: extract the `<script>` and run `node --check`; confirm every city `values` length == `params` length.
4. Record `sha256` of OLD (raw) and NEW (local). Generate a unified diff (`diff -u`, headers rewritten to
   `a/index.html` / `b/index.html`), base64 it.
5. In the Composio **remote workbench**: fetch the raw file, **assert its sha256 == the recorded OLD hash**,
   `patch -p1` the decoded diff, **assert result sha256 == the recorded NEW hash**. Do not proceed on mismatch.
6. Commit with `GITHUB_CREATE_OR_UPDATE_FILE_CONTENTS` (`owner: cfredrickson79-cmyk`, `repo: climate-haven-radars`,
   `path: index.html`, `branch: main`, `content:` = new HTML plaintext; it auto-base64s).
7. Wait ~60s, curl the live site with a cache-buster, and confirm live sha256 == the NEW hash.

(For a brand-new file instead of an edit, base64 the whole file and hash-gate it the same way — the hash assert is
the safety net either way. Never hand-paste large content without a hash check.)

## Current grades and where they come from
Comfort axes (Sunny, Storm, Flooding, Heat/Fire/Smoke, Allergy) are from Cameron's original data file.
**Water Access** and all Financials rows except Housing Affordability were researched Aug 2026:

- **Income tax (lower burden = higher):** UT flat ~4.45%, PA flat 3.07% (also exempts retirement income),
  NY progressive to 10.9%, WI progressive to 7.65%.
- **Property tax (city/county, lower = higher score):** Cache Co UT very low (~0.5%); Allegheny/Pittsburgh ~2.47%;
  Monroe/Rochester ~2.4–3%+; Dane/Madison ~1.8%.
- **Retirement tax-friendliness:** PA most friendly (no tax on retirement income); NY mixed; UT/WI moderate.
- **Cost of living:** Pittsburgh cheapest (~14% below US avg); Logan ~avg; Rochester ~+4%; Madison ~+3%.
- **Job market:** Madison strongest (capital+university, low unemployment); Pittsburgh large/diversified;
  Logan low unemployment but small; Rochester moderate.
- **Water Access (≤1 hr, kayak→jet ski):** Rochester 9 (Lake Ontario + Finger Lakes), Madison 9 (city lakes),
  Logan 8 (Bear Lake / Willard Bay / Hyrum / Pineview), Pittsburgh 7 (3 rivers + regional lakes).

Housing Affordability values (Logan 5, Pittsburgh 8, Rochester 8, Madison 5) are Cameron's own inputs — not researched.

## Likely next requests
- Add a city: Unlock → "+ Add city" in the app for a quick local try; for a permanent default, add a city object
  (with a Wikipedia `url`) to BOTH datasets in `defaults()`, bump `STORE_KEY`, redeploy.
- Add an axis (e.g. schools, winter mildness): add to `params` and append a value to every city; bump `STORE_KEY`.
- Refine Housing Affordability with researched numbers (currently Cameron's estimates).
