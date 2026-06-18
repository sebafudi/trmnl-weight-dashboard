# Weight Dashboard — TRMNL plugin

A full-screen (800×480, landscape) TRMNL e-ink plugin: dense weight stats on the
left, a long-term trend chart on the right. Built with [`trmnlp`](https://github.com/usetrmnl/trmnlp)
and deployed to TRMNL automatically on every push to `main`.

## Data model

The template reads two Liquid variables from the plugin's polling data source:

- **`rw`** — recent log, one semicolon-separated slot per **day** (`index 0 = today`, empty slot = no entry).
- **`hw`** — historical log, one slot per **week**, placed before the recent log in time. Already weekly averages.

## Chart

- **X-axis is real time.** Each half (history left, recent right, 50/50 split) maps points by elapsed days, not by index — so a year with one weigh-in and a year with forty take equal calendar width. Ticks sit on year boundaries (history) and month boundaries (recent).
- **History line** = robust LOESS (locally-weighted regression) through the weekly readings, sampled on a time grid: smooth, gap-aware, outlier-resistant.
- **Recent line** = causal time-aware EMA + raw weigh-in tracer (the headline stats are read off the EMA).

Tunable knobs live at the top of the `<script>` block in `src/full.liquid`
(`LOESS_SPAN`, `LOESS_ROBUST_ITERS`, `LOESS_GRID`, `HALF_LIFE_DAYS`, `SPLIT_PERCENT`, …).

## Local development

Requires Docker (or the `trmnl_preview` Ruby gem). From this folder:

```bash
bin/trmnlp serve   # → http://localhost:4567, live-reloads on save
```

Preview data is the dummy `rw`/`hw` in `.trmnlp.yml` — real weight history never lives in this repo.

## Deployment

Pushing to `main` triggers `.github/workflows/trmnl.yml`, which runs `trmnlp push --force`
to update the plugin identified by `id:` in `src/settings.yml`.

First-time setup:

1. Add a repository secret **`TRMNL_API_KEY`** (Settings → Secrets and variables → Actions).
2. Run `bin/trmnlp login` then `bin/trmnlp pull` once to overwrite `src/settings.yml`
   with the live plugin config (strategy, polling URL, etc.), and commit it — so CI
   deploys your real settings rather than the scaffold defaults.
