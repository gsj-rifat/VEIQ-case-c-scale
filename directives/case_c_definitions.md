# Case C — Q1–Q4 analysis definitions

Operational constraints for Phase 2 answers. Do not change headline metrics without updating this file and re-running `scripts/phase2_answers.py`.

## Budget

- Peak RSS ≤ 2048 MB (DuckDB `memory_limit=1800MB`)
- Wall time ≤ 300 s
- Prefer DuckDB over full pandas materialization

## Q1 Supply

- **Car key:** `lower(brand)|lower(model)|year|mileage_km|round(power_kw)|seller_ref`
- **On market:** `still_active` rows deduped by car key
- Exclude price and source from the key

## Q2 Liquidity

- Outcome: median days `first_seen → last_seen` among `status_raw = 'sold'`
- Drop `last_seen < first_seen`
- Always report `% still_active` (censoring)
- Primary segment: `seller_type`

## Q3 Depreciation

- Age = `2026 - first_registration_year`
- Filters: price €500–150k; year 2005–2026; age 0–15
- **Controlled for mileage:** `regr_slope(price, age)` inside mileage bins
- Headline band: 50–100k km

## Q4 Private vs dealer

- Comparable: `brand|model|year|fuel|body|mileage_bin` with ≥3 each side
- Gap: median over cells of `(med_private − med_dealer)`
