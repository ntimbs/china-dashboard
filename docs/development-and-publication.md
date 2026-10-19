# China dashboard development and publication

## Deploy to GitHub Pages

Live site: **https://ntimbs.github.io/china-dashboard/** (repository `ntimbs/china-dashboard`, forked from `giasmith/china-dashboard`).

First publish, from this folder on your Mac:

```bash
bash scripts/publish_to_github.sh            # or: DRY_RUN=1 bash scripts/publish_to_github.sh to rehearse
```

The script installs the two workflows into `.github/workflows/`, validates the data, makes the first commit (using
GitHub's no-reply address if you have no git email set), refuses to commit the private project inventory or any
`.env`, then creates the repo and pushes — with the GitHub CLI if it is installed and logged in, otherwise after
asking you to create the empty repo at github.com/new. If the first deploy job reports that Pages is not enabled,
set **Settings → Pages → Build and deployment → Source: GitHub Actions** and re-run the workflow.

After that, every push to `main` re-validates the data and republishes `site/`. All asset and data URLs are
relative, so the site works under the `/china-dashboard/` sub-path.

## Tests

```bash
pip install -r requirements-dev.txt
python3 scripts/validate_data.py
python3 -m unittest discover -s tests/py -v
python3 -m unittest discover -s pipeline/tests -v
node scripts/check_js.mjs && node --test
python3 -m playwright install chromium     # once
python3 -m http.server 8000 --directory site & python3 tests/browser/smoke.py
```

The browser smoke test loads every page at desktop and phone widths in light and dark mode, and fails on console
errors, horizontal scroll, or stray `null`/`undefined`/`NaN` text. On the map it switches tabs, filters, focuses a
province by clicking it (also with real mouse input, plus a drag that must not change the focus), opens a facility,
and asserts that there is no export control and no coordinate or profile link reaches the browser.

## Before you publish (checklist)

- [ ] Confirm that the DataCenterMap licence allows publishing facility names, operators and MW at province level; if not, rebuild with `--publish-names false`.
- [ ] Spot-check the hand-transcribed yearbook tables in `data_sources/` against the source images.
- [ ] Replace the China Briefing tariff compilation with the provincial 代理购电 notices (and fill Jilin, Gansu, Tibet, 蒙西).
- [ ] Re-check the remaining `medium`-confidence actuals (Xinhua / People's Daily reproductions) against the ministry pages when they become reachable.
- [ ] Add year-end 2025 figures if MIIT/NDA publish them (FP32 total compute, all-facility PUE, storage) so the `unresolved` 2025 targets resolve; add the NBS final 2025 digital-economy share (due ~Dec 2026).
- [ ] Set `meta.json` `data_status` to `live` only once every record is verified (the validator enforces this).
- [ ] Set a real contact in the pipeline's `DEFAULT_UA`, and complete the access review for each source in
      `pipeline/config/sources.yml` before enabling it.
