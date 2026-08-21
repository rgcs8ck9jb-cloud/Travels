# RafikTrip — handbook (v3)

Build reference for city guides in this project. Upload as project knowledge, replacing v2.

---

## 1. The site

| | |
|---|---|
| Repo | `travels`, public, GitHub Pages from `main` / root |
| Live | `https://<username>.github.io/travels/` |
| Home | `index.html` — static, reads `trips.json`, never edited |
| List | `trips.json` |
| Guides | `<city>.html` + `<city>-food.html` + `<city>-info.html`, flat in the root |
| Icon | `apple-touch-icon.png` (180×180) in the root |

Pages rebuilds in about a minute after a commit.

### Home page behaviour

- Groups by **month** (from `start`), then by **country** inside each month — always, even for a single city.
- Newest month first; inside a country, newest trip first.
- Badges: **Here now** (coral) on the most recent trip whose `start` has passed; **Upcoming** (teal) on future trips.
- Month and country groups are `<details>` accordions; open/closed state persists in `localStorage` under `rafiktrip:open`.
- Each card carries four real quick-links: Plan, Map (`#map`), Food, Practical — derived from `file`.
- Header shows guide count, country count, upcoming count.
- Invalid JSON produces a plain-English error card, not a blank page.

### trips.json schema

| Field | Req | Notes |
|---|---|---|
| `city` | yes | Card title |
| `country` | yes | Drives the country sub-heading |
| `dates` | yes | Free text, e.g. `"18–19 August 2026"` |
| `start` | yes | `YYYY-MM-DD`. Month grouping, sort order, badge logic |
| `file` | yes | Exact filename, case-sensitive |
| `hotel` | no | Shown under the city name |
| `tag` | no | Pill, e.g. `"12 stops · 2 mornings · 2 evenings"` |
| `blurb` | no | One sentence |
| `accent` | no | Hex for the card stripe. Default `#0F4C5C` |

---

## 2. Brand — RafikTrip

```
--teal #0F4C5C   --teal-d #0A3947  --teal-x #072B36
--teal-l #E4EEF1 --teal-l2 #F2F7F9
--coral #FF7A59  --coral-d #BF4526 --coral-l #FFEFE9
--ink #12262E    --ink2 #5A6E77    --ink3 #8DA0A8
--line #DFE7EA   --line2 #EDF2F4
--bg #FFFFFF     --surf #F5F7F8
--font "Manrope","Inter",-apple-system,…,Arial,sans-serif
```

Teal carries structure; coral marks accents only — evening blocks, active tab underline, Food/Practical chips, progress bar, "Here now" badge, warnings. `--coral-d` for coral text on white (raw coral fails contrast).

The monogram lives in `build.py` as `R_PATHS`, wrapped two ways: `ICON` (teal rounded square — favicon, footer, `apple-touch-icon.png`) and `MARK` (transparent, viewBox cropped to `16 12 32 42` so the R fills its box — headers on teal). The full lockup with wordmark and tagline appears only on the home page.

Tagline: **Tailored to you. Journeys that fit.**

## 3. Guide data model

Four globals, injected into the template at build time.

```js
const CITY = { name, country, dates, sub, plan, pills:[],
  hotel:{ name, short, gq, routeQ, lat, lng, alt } };

const DAYS = [{ slot:"Morning"|"Evening", day, dayShort, out, back, walk, after,
  title, intro, stops:[1,2,3], optional:[4], tips:[] }];

const STOPS = [{ n, name, short, alt, lat, lng, gq,
  dur, fee, hours, block, tips:[], more:`<p>…</p>` }];

const EXTRAS = [{ n:"A", name, short, alt, lat, lng, gq, dist, dur, fee, hours, more }];

const FOOD = { intro, dishes:[{name,note}],
  places:[{name,gq,type,area,price,note}], tips:[] };

const PRACTICAL = [{ title, warn?:true, items:[] }];
```

**Notes**

- `hotel.routeQ` is the string used as origin and destination in route URLs. Use the hotel's full name and street — far more reliable than coordinates.
- `short` is the label on the offline schematic. Keep it under ~14 characters.
- A stop may appear in more than one day block (e.g. a square seen lit at night and entered in the morning).
- `EXTRAS` may appear in a day's `stops`; route polylines skip them so a distant taxi site does not stretch the schematic.

---

## 4. Template mechanics

- **Route legs:** `[hotel, ...stops, hotel]` chunked into runs of 10 points, since Google caps a directions URL there.
- **Offline plan:** hand-built SVG, no library. Linear projection with `cos(lat)` longitude compression; greedy label placement over 40 candidate slots (8 directions × 5 radii) with collision boxes, canvas-edge rejection and leader lines; scale bar; hotel drawn as a black rounded square marked `H`.
- **Leaflet fallback:** if `typeof L === "undefined"`, relabel the toggle, switch to the offline plan, and stub `showOnMap`. The page must never depend on the CDN.
- **Evening blocks** draw coral; mornings draw teal. Same on both maps.

---

## 5. Research rules

- Search before writing. Prices, hours and closing days change every year.
- Cross-check ticket prices against two sources; give a range when they disagree.
- **Condé Nast Traveller / Traveler: blocked (403).** Say so; use Caravanistan, Advantour and current independent guides.
- **Coordinates are the weak point.** Nominatim and Overpass are blocked; Wikipedia is occasionally wrong. Pins are "accurate to roughly a block" — say it on the page, and route by name.
- Separate documented history from guide patter: "legend says", "historians treat this carefully".
- Check the season: heat, Ramadan, closing days, festivals.

---

## 6. Delivery checklist

- [ ] Hotel confirmed with the user; every route starts and ends there
- [ ] Day structure confirmed: early out, back for lunch/nap/pool, out again in the evening
- [ ] Rendered at 390×844, zero console errors
- [ ] All three tabs switch; `#map` and `#stops` deep links work; pin → Read → correct stop card
- [ ] Route buttons within the 10-point cap; hotel used by name
- [ ] Offline plan renders with no network; labels inside the canvas, no collisions
- [ ] Food page: 8–12 dishes, 6–8 places, one near the hotel
- [ ] Back / Home controls present on every page; no broken local links
- [ ] Brand: teal header, coral accents, Manrope, RafikTrip mark and favicon
- [ ] Prices marked indicative; Condé Nast limitation stated if food is included
- [ ] Filename lower case, no spaces; `trips.json` block supplied

---

## 7. Built so far

| City | File | Hotel | Structure |
|---|---|---|---|
| Bukhara | `bukhara.html` | Sahid Zarafshon Hotel | 19 stops, 3 extras · 2 mornings + 2 evenings · 15–17 Aug 2026 |
| Samarkand | `samarkand.html` | Khan Hotel Samarkand | 12 stops, 3 extras · 3 mornings + 3 evenings · 18–20 Aug 2026 |
| Tashkent | `tashkent.html` | not booked — anchored on Amir Timur Square | 10 stops, 3 extras · 2 mornings + 1 evening · Aug 2026 |

## 8. Candidates

Khiva completes the Uzbekistan trip; Tashkent needs re-anchoring once the hotel is booked. Baku (May 2026) could be retrofitted into the same format.
