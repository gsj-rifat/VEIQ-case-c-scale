# Case C: Q5 data quality SOP

Find ≥3 pathologies. For each: evidence count, which of Q1–Q4 breaks, handling.

## Required checks

1. **Placeholder prices**: `asking_price_eur <= 1` and `< 500` → breaks Q3/Q4 means; filter `< 500`
2. **Impossible years**: `1900`, `> 2026` → breaks Q3 age; filter `2005–2026`
3. **Inverted dates**: `last_seen < first_seen` → breaks Q2; drop from durations
4. **Relists**: same car_key sequential spells, especially prior `sold` → softens Q2; Q1 dedupe
5. **Null power_kw**: affects car_key collisions → flag Q1 under-merge risk

## Trust ranking after handling

1. Q4 (matched medians, filtered)
2. Q1 (deduped active supply; null-power caveat)
3. Q3 (within-band slopes after filters)
4. Q2 ranks OK; absolute days softer (censoring + relists)
