# KV Tervuren-Duisburg – stats site

Interactive stats page for the first team of KV Tervuren-Duisburg (stamnummer 384), home ground **Berg van Termunt**. It is hosted on GitHub Pages and shared with the team as a link, so it must work for people without any account or install, mostly on phones.

The current focus is **home vs away**: is it true that we take fewer points at home? Many more stats will be added over time.

## Current state

- Everything lives in one file: `index.html`.
  - d3 v7 (minified) is inlined in the first `<script>` tag.
  - The match data is a JSON array in `const DATA = [...]` in the second `<script>` tag.
  - All CSS is inline in `<style>`.
- There is no build step yet. GitHub Pages serves `index.html` from the `main` branch root.

### Suggested first refactor (do this before adding big features)

Split the single file into editable sources plus a build script that regenerates `index.html`:

```
data/wedstrijden.csv     # one row per match (source of truth)
src/template.html        # page with __D3__ and __DATA__ placeholders
vendor/d3.min.js         # d3 v7
build.py                 # reads CSV + template + d3, writes index.html
index.html               # build output, committed so Pages can serve it
```

Keep the output a single self-contained file. No CDN dependencies, so the page also works when opened offline or sent as an attachment.

## Data

One row per competition match of the first team. League matches only; cup matches (Beker van België) are excluded from the stats.

| column | meaning |
|---|---|
| `seizoen` | season, e.g. `2025-26` |
| `datum` | ISO date `YYYY-MM-DD`, empty if unknown |
| `venue` | `Thuis` (Berg van Termunt) or `Uit` |
| `tegenstander` | opponent name |
| `voor` | goals scored by Tervuren-Duisburg |
| `tegen` | goals conceded |
| `uitslag` | `W` (won), `G` (draw), `V` (lost), from our perspective |

Rules:

- `voor`/`tegen` are always from Tervuren-Duisburg's perspective, regardless of venue.
- `uitslag` must agree with `voor` vs `tegen`. Validate this on every build.
- Leagues:
  - 2024-25: 2e prov. B Vlaams-Brabant. Final: 6th, 30 played, 13-6-11, 68–44, 45 pts.
  - 2025-26: 2e prov. B Vlaams-Brabant. Champions: 30 played, 18-3-9, 81–45, 57 pts.
  - 2026-27: 1e prov. Vlaams-Brabant, ongoing.
- After adding matches, check season totals against the official table where possible.

Sources for results:

- voetbalinbelgie.be: blocks scripts, check manually.
- voetbalexpress.be: Brabant 1e provinciale pages, includes per-team calendars.
- voetbalkrant.com: incomplete for lower divisions.

Results Naten provides directly take precedence over these sites.

## Definitions (keep consistent)

- Points: win = 3, draw = 1.
- Win % = wins / matches played.
- Points per match = (3·W + G) / matches.
- Goals per match = goals / matches.
- Clean sheet = match with `tegen == 0`.

## UI conventions

- **Language: Dutch (Flemish)** for everything on the page. Decimal comma (`1,83`), dates `dd/mm/yyyy`.
- Colors (CSS variables in `:root`, with a dark-mode override):
  - home `--home` (blue), away `--away` (orange)
  - win `--win` (green), draw `--draw` (grey), loss `--loss` (red)
- Keep it understandable for teammates and parents, not analysts:
  - chart titles only, no explanatory paragraphs
  - values printed on bars
  - hover tooltips
- Mobile first. Test at ~390px wide; charts are drawn from `clientWidth` and redraw on resize.
- The season filter (`Alle seizoenen` / per season) must drive every section.
- Print and "Save as PDF" should stay usable: unselected filter buttons hide, tables expand.

## Checking changes

Before committing, open the built file in a headless browser at desktop (1200px) and mobile (390px) widths. Confirm there are no console errors and that the numbers render.

Playwright for Python is a quick way to do this: screenshot both widths and check for page errors.

## Deploying

Commit and push to `main`. GitHub Pages republishes within a minute or two at the same link.

The repo is public, so never commit anything private, such as player contact details or medical info.
