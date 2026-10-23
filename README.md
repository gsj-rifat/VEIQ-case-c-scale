# VEIQ Case C: Scale

Five market questions on 1,000,000 Dutch car listings, answered under a hard budget of **2 GB RAM and 5 minutes**. The analysis runs in DuckDB directly over the parquet file and finishes in about **9 seconds using about 106 MB** of memory.

This is my solution to the VEIQ AI internship take-home (case C). The brief is in [`VEIQ-case-C-scale.pdf`](VEIQ-case-C-scale.pdf) and the one-page answer is in [`WRITEUP.md`](WRITEUP.md).

## Results

| Question | Answer |
| -------- | ------ |
| 1. Supply | **360,007** distinct cars are on the market (360,091 active listings); 981,920 distinct cars all-time |
| 2. Liquidity | **Private sellers** sell fastest (26 days median vs 30 for dealers); about 36% of listings are still active (censored) |
| 3. Price by age | At 50-100k km, price drops per year of age: diesel **-€2,271**, hybrid -€1,832, electric -€1,576, petrol **-€1,459** |
| 4. Who is cheap | Private sellers undercut dealers by **€505 (-7.0%)** on 5,247 comparable cells |
| 5. What is wrong | 5 issues found: €1 placeholder prices, impossible years, inverted dates, relists, null `power_kw` |

Each answer states its definition and where the uncertainty sits. Question 5 also says which of Q1 to Q4 each problem affects and how it was handled.

## How I approached it

- **Stack:** DuckDB queries over the parquet file, with no full pandas load. A full pandas load alone is about 750 MB.
- **Q1:** a car is identified by brand, model, registration year, mileage, power and seller. Price and marketplace are left out of the key. Most duplicates turned out to be relists on the same site, not cross-posts.
- **Q2:** median days to sold, always shown next to the share of listings still active.
- **Q3:** "controlled for mileage" means the age slope is estimated inside a mileage band.
- **Q4:** private and dealer prices are compared only inside cells of identical brand, model, year, fuel, body type and mileage band.
- **Q5:** every issue was checked for whether it actually changes a headline number.

## Repository layout

| Path | What it is |
| ---- | ---------- |
| `case_c_analysis.ipynb` | Main notebook with Q1 to Q5, executed |
| `scripts/full_answers.py` | Same analysis as one script; also measures and asserts the time and memory budget |
| `WRITEUP.md` | One-page write-up: numbers, definitions, peak memory, runtime, use of AI |
| `directives/` | The definitions used for Q1 to Q4 and the checklist used for Q5 |
| `outputs/` | JSON results written by the scripts |
| `deliverables/` | Standalone copy of the submission files |
| `case-c_market.parquet` | The dataset supplied with the brief (1M synthetic listings) |
| `requirements.txt` | Pinned Python dependencies |

## Run it

```bash
python -m venv .venv
.venv\Scripts\activate        # macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
python scripts/full_answers.py   # prints all answers and writes outputs/full_answers.json
```

Run from the repository root so the parquet path resolves. To use the notebook instead, run `jupyter notebook case_c_analysis.ipynb` and pick a kernel from the environment above.

Tested with Python 3.11 and DuckDB 1.5.5. No API keys or external services are needed.

## Data

`case-c_market.parquet` was supplied by VEIQ AI with the case brief. It is synthetic data generated to resemble the Dutch used car market, so the cars are not real.
