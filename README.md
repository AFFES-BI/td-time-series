# td-time-series

TD4 (BI) — real atmospheric CO2 time series for Le Mans from the Open-Meteo Air Quality API:
collection, cleaning, smoothing (rolling mean, exponential), and a short linear projection,
published as a dashboard on GitHub Pages.

- `scripts/fetch_series.py` — pulls 90 days of hourly CO2, aggregates to daily, writes `data/series.csv`
- `scripts/analyze_series.py` — cleans, smooths, projects, writes `docs/data/series_analysis.json`
- `.github/workflows/update_series.yml` — runs both weekly (Monday 06:00 UTC) or on demand
- `docs/index.html` — the dashboard
- `REFLECTIONS.md` — written answers to the TD's reflection questions
