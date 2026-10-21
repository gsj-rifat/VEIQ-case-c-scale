# VEIQ Case C — submission pack

Per the brief: **one script or notebook that produces all five answers**, plus a **one-page write-up** with numbers, peak memory, and runtime.

| File | Role |
|------|------|
| `WRITEUP.md` | One-page write-up (answers + peak RAM + runtime) |
| `case_c_analysis.ipynb` | Notebook producing Q1–Q5 |
| `full_answers.py` | Equivalent one-shot script (same answers) |
| `full_answers.json` | Machine-readable run output (evidence of measured budget) |
| `requirements.txt` | Dependencies |

## How to reproduce

Place `case-c_market.parquet` in the **parent** folder (`../case-c_market.parquet`), then:

```powershell
pip install -r requirements.txt
python full_answers.py
# or open case_c_analysis.ipynb and Run All
```

Expected budget (measured): **~8.8 s**, **~106 MB** peak RSS.
