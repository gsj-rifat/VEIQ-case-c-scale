# VEIQ Case C: one-page write-up

**Peak memory:** 105.9 MB RSS · **Runtime:** 8.83 s · (DuckDB `memory_limit=1800MB`)  
**Stack:** DuckDB over `case-c_market.parquet` (no full pandas load) · reproduce: `python scripts/full_answers.py` or see `deliverables/`

---

### Q1: Supply
**Definition:** A *car* = `lower(brand)|model|first_registration_year|mileage_km|round(power_kw)|seller_ref`. *On the market* = `still_active` rows after that dedupe. Price and marketplace (`source`) are excluded from the key (price moves; cross-posts share the car).

**Answer:** **360,007** distinct cars currently on market (360,091 active listings). All-time unique cars: **981,920** of 1,000,000 listings. Duplicate pairs are mostly sequential relists (~17.9k), not concurrent cross-site copies (~210).

---

### Q2: Liquidity
**Definition:** Median days `first_seen → last_seen` among `status_raw='sold'` only; drop inverted dates; always report `% still_active` (~36% censored). Do not average only finished listings.

**Answer:** **Private sellers are fastest (26d median to sold)** vs dealers **30d**. Cheaper bands sell faster (`<10k` 28d → `35k+` 33d). Fuel types are essentially flat (~29d). **Uncertainty:** some “sold” spells later relist (Q5#4), so treat absolute days as directional, not causal. **Limitation:** the median is taken over sold listings only (about 7% of rows; 56% are `removed`), so it is conditional on a listing finishing as sold. A survival (Kaplan-Meier) comparison that uses all listings would be the next step.

---

### Q3: Price by age (per fuel), controlled for mileage
**Definition:** Age = `2026 − first_registration_year`. Filters: price €500–150k, year 2005–2026. *Controlled for mileage* = linear age slope of asking price **inside** the 50–100k km band (`regr_slope`), not a raw age–price correlation (mileage confounds age).

**Answer (€ / year of age @ 50–100k km):** diesel **−2,271** · hybrid **−1,832** · electric **−1,576** · petrol **−1,459**. Diesel depreciates fastest in this band; petrol slowest.

---

### Q4: Private vs dealer
**Definition:** Comparable cell = `brand|model|year|fuel|body|mileage_bin` with ≥3 private and ≥3 dealer (same filters as Q3). Gap = median over cells of `(median_private − median_dealer)`.

**Answer:** **Yes, private undercuts by €505 (−7.0%)** on **5,247** cells; private cheaper in **~90%** of cells. The direction holds for cell-size thresholds 1 to 10; the size moves from €463 to €580. Unmatched overall medians are *not* the answer (mix differs).

---

### Q5: What’s wrong (≥3), evidence, impact

| # | Issue | Evidence | Breaks | Handling |
|---|--------|----------|--------|----------|
| 1 | €1 / placeholder prices | **3,981** ≤ €1; **7,860** < €500; raw mean ~€100 below cleaned mean | Q3, Q4 (means) | Excluded price < €500; prefer medians |
| 2 | Impossible years | **968** year=1900; **1,943** year 2027–28 → negative ages | Q3, Q4 | Filter year to 2005–2026 |
| 3 | Inverted dates | **4,284** with `last_seen < first_seen` | Q2 | Dropped from duration stats |
| 4 | Relists after “sold” / short spells | **18,078** sequential same-car pairs; **2,144** prior spell marked sold | Q2 (reliability), Q1 (listing≠car) | Q1 car-key dedupe; flag Q2 sold as uncertain |
| 5 | Null `power_kw` | **279,355** (~28%) → key uses `'NA'` | Q1 (possible under-merge) | Flagged; key still best fingerprint |

**Trust stance:** Q1 and Q4 are the most trustworthy after filters. Q3 slopes are trustworthy *within* the mileage band and year/price filters. Q2 ranks (private > dealer; cheap > expensive) are trustworthy; absolute day levels are softer because of censoring and relists.

---

### Optional: 100× scale (11.4M rows)
Keep aggregates in DuckDB/SQL over partitioned parquet; never materialize full frames; pre-aggregate car keys and price/age bins; push filters early; write only summary tables for plots.

---

### Use of AI

I used Cursor to write and iterate on the DuckDB/SQL and pandas glue code (boilerplate, syntax, query drafts). The decisions were mine, checked against the data, not generated:
- **Q1:** tested 4 candidate dedupe keys before picking one. Including price collapsed distinct cars to 999,997 (proving price had to be excluded), and dropping `seller_ref` over-merged different sellers' identical cars.
- **Q3:** price correlated with both age (−0.75) and mileage (−0.65) at once, so a pooled age slope is confounded, so I split it into mileage bins myself and confirmed the depreciation rate itself changes by band before calling it "controlled for mileage."
- **Q4:** ran the €505 gap through a min-cell-size sweep (1/3/5/10 listings/side) to rule out a knife-edge threshold before trusting it.
- **Q5:** every pathology count, and whether it actually moved a headline number (e.g. relists *not* moving the Q2 median), was a query I ran and read myself.
