# China Dashboard

A static dashboard (no build step, no runtime dependencies) modelled on the
[U.S. Data Center Ecosystem Tracker](https://ntimbs.github.io/us-data-center-dashboard/): a **province map of China's
data-center inventory** with layer tabs for status, capacity, installed power, 2024 generation, electricity cost and
water, plus two secondary pages that compare **plan targets (five-year plans and computing-power action plans) with reported results** and catalogue the
**legislation** behind them. Designed to be published on GitHub Pages.

> **Location rule.** The inventory merges a licensed DataCenterMap export with public crawls. It is published at
> **province level only**: `site/data/facilities.json` holds no coordinates, addresses, cities or profile links, the
> build script strips them, the validator and the browser test both assert their absence, and the map draws one
> bubble per province. Map tabs are shown only for layers that have data (the U.S. tracker's hazard and
> opposition tabs are left out until a source exists).

## Run it

```bash
python3 -m http.server 8000 --directory site      # then open http://localhost:8000/
```

Opening `site/index.html` straight from disk will not work: browsers block `fetch()` and ES modules on `file://`.
The page tells you so if you try.

## Layout

```
site/                     the published site (this folder is what GitHub Pages serves)
  index.html              the map dashboard;  plan.html, legislation.html  the secondary pages
  assets/css/style.css    design tokens (light/dark), layout, components;  dashboard.css  the map page
  assets/js/lib/          pure logic + small UI helpers   <- unit-tested (inventory.js = map filtering/binning)
  assets/js/pages/        one module per page (dashboard.js, plan.js, legislation.js)
  data/                   meta.json, provinces.json, facilities.json, power_plants.json, power_lines.json,
                          plan_targets.json, plan_actuals.json, legislation.json
data_sources/             hand-transcribed official tables that feed provinces.json (see data_sources/README.md)
schemas/                  JSON Schema for every data file (the contract the build script must meet)
scripts/build_dashboard_data.py   builds the three map files from the EDWC research folder (see Data below)
scripts/build_power_lines.py      builds power_lines.json from the OpenStreetMap GeoPackage (see Data below)
scripts/validate_data.py  schema + cross-file checks (referential integrity, derived values, privacy rule, dates ...)
pipeline/                 data-gathering pipeline (stub: polite fetcher, source registry) - see pipeline/README.md
tests/js  tests/py  tests/browser   unit tests and a Playwright smoke test
.github/workflows/        validate.yml (PRs) and pages.yml (deploy on push to main)
```

## The map dashboard

Layout follows the U.S. tracker: filters on the left (search, project status, facility type, reported capacity,
inventory source; there is deliberately no CSV export, the inventory is view-only), KPI tiles, a tab
strip of map layers, a legend that also carries the source attribution for the active layer, a profile chart (donut
for the selection, ranked bars for province comparisons) and a detail panel on the right. The map is a plain SVG (Albers equal-area outlines pre-projected by the
build script), so there is no tile server, no map library and nothing loaded from a CDN.

There is no province dropdown: **clicking a province on the map focuses the dashboard on it.** The KPI tiles, the
sidebar counts and the profile card narrow to that province (on the Generation and Power tabs the card becomes its
power infrastructure profile, GEM units or installed capacity by technology; on the comparison tabs its bar is
highlighted, with its rank, even outside the top eight), and the Generation tab draws only its plant units. Other
provinces dim but keep their bubbles and stay clickable. Clicking the province again, clicking empty map, Close or
Reset returns to all of China; dragging the map never changes the focus.

| Tab | Province shade | Points / bubbles | Source |
|---|---|---|---|
| Status | facility count in the selection | one status wheel per province, sized by count | inventory |
| Capacity | known MW in the selection | bubble sized by known MW | inventory |
| Power | installed MW of GEM units (choose a technology) | pale count bubbles; transmission lines ≥250 kV | Global Energy Monitor; OpenStreetMap (ODbL) |
| Generation | 2024 generation (NBS yearbook 9-19) | GEM plant units by technology; transmission lines ≥250 kV | NBS + GEM; OpenStreetMap (ODbL) |
| Electricity cost | 1–10 kV industrial & commercial rate, May 2025 | pale count bubbles | China Briefing compilation (tier C) |
| Water scarcity | per-capita resources, use÷resources, totals, industrial use (2024) | pale count bubbles | MWR bulletin via NBS yearbook 8-8 / 8-9 |
| Policy | EDWC national-hub provinces | pale count bubbles | NDRC approvals; legislation topics in the chart |

The URL hash carries the active tab (`#view=cost`), so a view can be linked to.

