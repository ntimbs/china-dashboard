# China dashboard data and methodology

## How a target's status is computed

`site/assets/js/lib/status.js` is the single source of truth. Three kinds of target:

| kind | meaning | status rule |
|---|---|---|
| `level` | reach a value relative to a baseline by a year | `progress = (latest - baseline) / (target - baseline)`, compared with the share of the baseline-to-target period that has elapsed |
| `threshold` | stay at or above (or below) a value, e.g. a grain-output floor | met or not at the latest year |
| `rate` | average annual rate over the plan period, e.g. GDP growth | average of reported years vs the target |

Statuses: **achieved**, **on track** (within 10 percentage points of the expected pace, `statusTolerance` in
`config.js`), **behind pace**, **missed** (a full-year figure for the target year falls short), **unresolved** (the
target year has passed, judged by `meta.as_of`, but no full-year figure for it has been published), **no data**,
**invalid** (the target definition cannot be evaluated; the reason is shown).

Two rules keep the scoring honest. An actual may carry an `as_of` date; anything other than 31 December is a
*partial-year* figure, which is drawn hollow, counts only the elapsed share of its year towards the expected pace,
can confirm an early achievement, but can never settle a deadline. And values reported *after* the target year are
plotted for context but never count towards meeting a dated target (a 2025 target is not achieved by a 2026 value).

Why `threshold` exists: "at least 650 Mt" with a 2020 baseline of 669 looks like a *decrease* if modelled as a level
target, and every good year would score as "behind". The validator rejects that mistake.

## Data

All data lives in `site/data/` and is checked by `python3 scripts/validate_data.py` (also run in CI; a failing
validation blocks deployment). Plan targets, actuals and legislation records carry `placeholder` / `verified` flags. Setting `meta.json` `data_status` to `"live"` makes the validator reject any placeholder or unverified
record. The dataset is currently `partial` (compiled 2026-10-04).

**Source rule.** Values come only from official primary sources — government documents, NBS/NHC/MOHRSS statistical
communiqués and bulletins, ministry briefings, 数字中国发展报告 — or from Xinhua / People's Daily reproductions of
them (tagged `medium`). Secondary estimates (CAICT white papers, trade press, academic papers) are never stored as
values; where they are the only figure in circulation they are mentioned in a `note`. Figures the source prints as a
floor or ceiling (超过 / 以上 / 不超过) carry `qualifier: "floor" | "ceiling" | "approx"` and are shown as ≥ / ≤ / ≈;
the status engine scores the printed value. No value is derived: five-year cumulative reductions are recorded only
when an official document prints them, never compounded from annual rates.

**Plan targets and actuals** (`plan_targets.json`, `plan_actuals.json`): every target carries a `theme` that places it in
the data-center story (compute · energy/PUE/carbon · digital demand · AI/innovation · macro context); the `themes`
block names the map tab and legislation topics each theme links to, and the plan page is grouped by it. The
headline indicator tables of the 14th
(2021–2025) and 15th (2026–2030) Five-Year Plan outlines (the 15th read from the NDRC PDF, whose table is text; the
gov.cn copy is an image), the 14th FYP Digital Economy plan, the four data-center / computing-power action plans
(MIIT 2021, MIIT 2023, NDRC EDWC opinions 2023, green data-center plan 2024), the ICT industry 15th FYP (MIIT PDF,
工信部规〔2026〕188号) and the MOST national AI pilot-zone guideline (国科发规〔2019〕298号; 17 zones by Dec 2021 vs
"about 20" by 2023, Xinhua count). Each actual carries `source.url`, a verbatim `quote`, `retrieved` and `confidence`. Target
`note` fields record modelling choices and definition changes — read them before quoting a status: energy intensity
switched to an adjusted basis in 2023; computing power is FP32 to 2024 and FP16 智能算力 from 2025; the national PUE
series changed population in 2025; 5G 普及率 is per 100 population, not a share of mobile users. Known gaps with no
official figure: total FP32 EFLOPS and all-facility PUE for end-2025, 智能算力占比, 先进存力占比, industrial-internet
platform penetration, the hub green-power share, and a point value for the pension participation rate (floor only).

**Legislation** (`legislation.json`): 94 national laws, regulations, departmental rules, standards, plans and policy
notices on the digital economy — data governance and security, cybersecurity, AI, platform economy, e-commerce,
fintech, digital government, computing power and data-center energy use — with document numbers,
adoption/publication/effective dates and a short summary. `verified: true` (93 of 94) means the dates were read from a
government page; the one exception (the 2018 等级保护条例 draft) says why in `note`. `date_precision: month` means
the issuing day is not published; the record then carries the first of the month and the release date in
`date_published`.

**Province outlines and indicators** (`provinces.json`, built by `scripts/build_dashboard_data.py`): 31 provinces +
Hong Kong and Macau, with SVG paths, a label anchor, the East Data West Computing hub each belongs to, and these
indicators: 2024 generation and hydro generation (NBS yearbook table 9-19), 2020/2024 electricity consumption (9-14),
2024 water resources and per-capita resources (8-8), 2024 water use incl. industrial (8-9), the May 2025 1–10 kV
industrial-and-commercial tariff (China Briefing compilation, tier C — Jilin, Gansu, Tibet and the SARs are not in
that source and Inner Mongolia shows the eastern grid only), and installed MW by technology summed from the GEM
units inside each outline. Two values are computed and labelled as such in the UI: generation − consumption, and
water use ÷ water resources. The transcribed source tables live in `data_sources/` with their URLs; the yearbook
publishes them as images, so they were typed by hand and should be spot-checked before citation.

**Facilities** (`facilities.json`): 833 records merged from the DataCenterMap export (licensed; 459), the baxtel
crawl (108), the DCKB sample (11), the two UVA geojsons (hyperscale 36, western 44) and 307 datacenters.com listings
(see below), de-duplicated on source id, normalised name, and proximity (unnamed records within 400 m of a named one). Each record keeps name, operator,
province, status (Operating / Development / Proposed / Unknown), facility type, reported MW and capacity class,
PUE and online year where the source had them, and the list of sources. Twenty DataCenterMap rows have no
coordinates in the export and were placed by their market name. Three rows fell outside every outline (one in
Taiwan, which the map does not cover). Run with `--publish-names false` to anonymise names and operators.

The datacenters.com rows (`data_sources/datacenters_com_sites.csv`, source key `dccom`) are China listings that were
compared with this inventory on 15 Oct 2026 and approved one by one: listings already in the inventory, resellers and
cloud regions, and uncertain matches were left out. The directory gives no status, so these records are entered as
Operating with facility type Unknown and no MW; they carry no coordinates and are placed by province. Where the
inventory holds only one city-level record for an operator (for example "China Telecom Shanghai") and the directory
names several buildings, at most one of those buildings can be that record, so such cities may be over-counted by one.

`data_sources/facility_details.csv` adds figures that the inventory sources do not carry (standard racks, computing
power, AI accelerator cards, building area, PUE) to individual facilities. Each row names its basis (for example
"vendor-reported" or "first-phase design") and its sources, and the facility profile on the map shows both. These
figures are not used in the MW totals or capacity classes.

**Power plants** (`power_plants.json`): 16,284 Global Energy Monitor units (coal, hydro, wind, solar, nuclear,
bioenergy) projected to the view box with their province. Status was filtered to operating upstream in the IGARSS
workflow; several units of one plant share a point. Gas, oil and waste plants came from OpenStreetMap in that
workflow with capacities in kW, so they are excluded here.

**Transmission lines** (`power_lines.json`, built by `scripts/build_power_lines.py`): OpenStreetMap `power=line`
ways from the November 2025 China extract (`_SCSP_export_large/EDWC_China_GIS/04_grid_transmission_osm/`), kept when
`max_voltage` is at least 250 kV. China has no voltage class between 220 and 330 kV, so this means 330 kV and up
(330/500/750 kV AC, ±400 to ±1100 kV DC, 1000 kV UHV AC); every 220 kV line is left out. Four classes are drawn, by
colour and width: 330–400 kV, 500 kV, 660–750 kV and UHV ≥800 kV. Result: 22,284 ways, about 292,600 km of mapped
route (500 kV 183,800 · UHV 42,600 · 330–400 kV 35,200 · 660–750 kV 31,100). Read the km as a lower bound on route
length, not as official circuit length: 30% of OSM power lines carry no voltage tag and are dropped, and parallel
lines mapped as separate ways count separately. Ways tagged construction or disused (93) are excluded. Lines are
simplified to about 1 km, split at province borders (each segment goes to the province holding its midpoint) and
dimmed outside a focused province; the paths never take pointer events, so province clicks pass through them.
**The GeoPackage stores every power line twice** (22,376 OSM ids appear twice with identical tags and geometry, and
the same is true of `01_working/data/working/osm_filtered.gpkg`); the build de-duplicates by OSM id, but any
analysis that sums lengths straight from either file double-counts. OSM data is © OpenStreetMap contributors under
the ODbL; `power_lines.json` is a derived database under the same licence, and the map legend carries the credit.

Rebuild the map files from the EDWC folder (this repository sits inside it):

    python3 scripts/build_dashboard_data.py --edwc-root .. --check
    python3 scripts/build_power_lines.py --edwc-root ..            # after provinces.json; --min-kv 220 to add 220 kV

