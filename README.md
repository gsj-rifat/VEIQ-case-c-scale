# VEIQ Case C — Scale

Internship take-home: five market questions on `case-c_market.parquet` under a **2 GB RAM / 5 min** budget.

## Layout

| File | Role |
| ---- | ---- |
| `case-c_market.parquet` | 1M synthetic Dutch listings |
| `VEIQ-case-C-scale.pdf` | Brief |
| `case_c_analysis.ipynb` | Main analysis notebook (Q1–Q5) |
| `WRITEUP.md` | One-page submission write-up |
| `scripts/full_answers.py` | Reproducible Q1–Q5 + budget |
| `requirements.txt` | Python deps |
| `directives/` | Locked definitions + Q5 SOP |

## Setup

```bash
pip install -r requirements.txt
jupyter notebook case_c_analysis.ipynb
```

Run from the repo root so the parquet path resolves.

**Note:** Prefer DuckDB here. Full pandas load is ~750 MB; Polars may crash on CPUs without AVX2 — use `polars[rtcompat]` only if you switch stacks.

Deliverables for the brief are gathered in [`deliverables/`](deliverables/) (notebook + script + one-page write-up).

## Workflow — complete (Q1–Q5 + write-up)

```bash
pip install -r requirements.txt
python -m ipykernel install --user --name veiq-case-c --display-name "Python (VEIQ case study)"
python scripts/full_answers.py          # → outputs/full_answers.json (~9s, ~106 MB)
# or: jupyter notebook case_c_analysis.ipynb
```

**Notebook kernel:** select **Python (VEIQ case study)** — the notebook is pinned to the project `.venv` where `duckdb` is installed. If imports fail, you are on the wrong kernel (e.g. conda `base`).

**One-page submission:** [`WRITEUP.md`](WRITEUP.md)

| Q | Headline |
| - | -------- |
| 1 | **360,007** distinct cars on market; 981,920 all-time |
| 2 | **Private** fastest (26d vs dealer 30d); ~36% censored |
| 3 | @50–100k km: diesel **−€2,271**/yr … petrol **−€1,459**/yr |
| 4 | Private undercuts by **€505 (−7.0%)** on 5,247 cells |
| 5 | 5 issues: €1 prices, bad years, inverted dates, relists, null power |

Definitions: `directives/case_c_definitions.md` · Q5 SOP: `directives/case_c_q5.md` 

## Deliverables (from the brief)

- One script/notebook that prints all five answers  
- One-page write-up: numbers, definitions, **peak RAM**, **runtime**  
- Say what AI generated vs what you verified  

## External config

None — analysis is local over the provided parquet. No API keys.
