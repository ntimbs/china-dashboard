# China Data-Center Ecosystem Tracker

The China Data-Center Ecosystem Tracker is a province-level companion to the [U.S. Data Center Ecosystem Tracker](https://ntimbs.github.io/us-data-center-dashboard/). It brings together data-center inventory with energy, water, computing-policy, and legislative context to help researchers explore the infrastructure behind AI in China.

**Live dashboard:** https://ntimbs.github.io/china-dashboard/

## Dashboard views

- **[Province map](https://ntimbs.github.io/china-dashboard/):** explore facility status and reported capacity alongside power infrastructure, 2024 generation, industrial electricity costs, water measures, and East Data West Computing policy. Select a province to focus the map and summary cards.
- **[Plan targets](https://ntimbs.github.io/china-dashboard/plan.html):** compare national Five-Year Plan and computing-power action-plan targets with reported results. Target statuses account for partial-year figures and unresolved deadlines.
- **[Legislation](https://ntimbs.github.io/china-dashboard/legislation.html):** browse the national laws, regulations, standards, plans, and policy notices that shape China's digital economy and data-center development.

The facility inventory is displayed at province level. The public map does not provide facility coordinates, addresses, profile links, or a downloadable inventory. Province totals and nearby infrastructure are context for exploration; they do not establish facility-level effects or grid service.

## Data and documentation

The dashboard currently labels its dataset **partial** (data as of 12 October 2026). Sources and caveats are shown with the relevant views. The project documentation remains available in focused guides:

- [Dashboard guide](docs/dashboard-guide.md) — map layers, interactions, local setup, and repository layout.
- [Data and methodology](docs/data-and-methodology.md) — source rules, inventory and contextual datasets, and how target status is calculated.
- [Development and publication](docs/development-and-publication.md) — deployment, tests, and the [open publication checklist](docs/development-and-publication.md#before-you-publish-checklist).
- [Research notes](docs/research/2026-10-04/README.md) — source-verification working records.

The publication checklist still calls for review of the commercial inventory licence and spot-checks of hand-transcribed source tables. Consult the underlying source records before citing a figure or target status.

## Run locally

Serve the `site/` directory from the repository root, then open http://localhost:8000/:

```bash
python3 -m http.server 8000 --directory site
```

## Credit

This repository is [Nathan Timbs's fork](https://github.com/ntimbs/china-dashboard) of [Gia Smith's China dashboard](https://github.com/giasmith/china-dashboard). The detailed project notes in `docs/` preserve Gia's original documentation.
