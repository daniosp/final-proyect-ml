# Clearview Properties — Residential Price Analysis

Final project for the *Aprendizaje Automático* (Machine Learning) course, MSc in Engineering Sciences, EAFIT (2026-2).

The project answers a consulting brief (`task_info/Brief_Candidate_DataScientist.docx`) for a real estate agency
that wants to support its brokers with evidence from its own historical sales:

| # | Business question | Approach (summary) |
|---|---|---|
| Q1 | What is a reasonable listing **price range** for a property? | Price model with prediction intervals |
| Q2 | Which characteristics **drive price**, split into owner-modifiable vs structural/location? | Model-based importance and effect sizes |
| Q3 | Which improvements are a **profitable investment**? | What-if uplift simulation, break-even cost, external cost benchmarks |
| Q4 | Are there **market segments** that behave differently? | Clustering and supervised segmentation |

> Status: **All phases (0–7) and the client-facing executive deck are complete on branch `ACC`.**

## Progress

| Phase | Notebook section | Status |
|---|---|---|
| 0. Reproducible environment | 0 | Done |
| 1. Data quality audit and cleaning | 1 | Done (incl. `Garage Cars = 99` repair) |
| 2. Exploratory analysis and feature engineering | 2 | Done (`DROP_LOT_FRONTAGE` confirmed via CV) |
| 3. Q1 — Price range model (Blend + 80% CQR intervals) | 3 | Done |
| 4. Q2 — Price drivers (Structural/Location vs Modifiable) | 4 | Done |
| 5. Q3 — Improvement ROI (counterfactuals vs cost benchmarks) | 5 | Done |
| 6. Q4 — Market segmentation (K-Means + PCA) | 6 | Done |
| 7. Conclusions, deck & summary | 7 | Done (`main/Clearview_Properties_Executive_Deck.pdf`) |

## Repository structure

```
proyecto-final-ml/
├── data/
│   ├── Data.csv                 # raw transactions as provided by the client (never modified)
│   ├── Data_Dictionary.txt      # variable descriptions and codes
│   └── processed/               # written by the notebook (git-ignored, regenerated on every run):
│       ├── data_quality_log.csv  #   every issue found, rows affected, action taken and rationale
│       ├── clean_all_sales.csv   #   all valid sales after repairs
│       └── analysis_sample.csv   #   sales used by the price models (Normal + new homes, <= 4,000 sqft)
├── main/
│   ├── clearview_analysis.ipynb               # technical notebook: full end-to-end workflow
│   ├── Clearview_Properties_Executive_Deck.pdf # 11-slide non-technical client presentation
│   └── figures/                               # charts exported by the notebook
├── task_info/                   # project brief
├── pyproject.toml               # dependencies
├── uv.lock                      # exact pinned versions (reproducible environment)
└── .python-version              # Python 3.12
```

The client-facing presentation (`main/Clearview_Properties_Executive_Deck.pdf`) translates all findings into business language across 11 slides.

## Setup

Requirements: [uv](https://docs.astral.sh/uv/getting-started/installation/) (tested with uv 0.12). uv installs the right
Python version (3.12) automatically.

```bash
git clone <repo-url>
cd proyecto-final-ml
uv sync --locked   # creates .venv with the exact versions from uv.lock (fails if the lock is out of date)
```

**Adding a dependency:** use `uv add <package>`, which updates `pyproject.toml` *and* `uv.lock` together, and commit
both files. Editing `pyproject.toml` by hand leaves the lock out of date, so `uv sync --locked` fails.

> If `uv` fails with `invalid peer certificate: UnknownIssuer` (common behind corporate proxies or antivirus TLS
> inspection), tell it to trust the operating system certificate store:
> `uv sync --system-certs`, or set the environment variable `UV_SYSTEM_CERTS=1`.

## Running the analysis

Interactive:

```bash
uv run jupyter lab main/clearview_analysis.ipynb
```

In VS Code, open the notebook and select the `.venv` interpreter (`.venv/Scripts/python.exe` on Windows,
`.venv/bin/python` on macOS/Linux) as the kernel.

Always run the notebook in this environment. Its first code cell stops with an error if the kernel is not Python 3.12,
and it prints the versions of the main libraries.

Headless, end to end (the notebook is reproducible with *Restart & Run All*, about 5 minutes on a laptop):

```bash
uv run jupyter nbconvert --to notebook --execute --inplace main/clearview_analysis.ipynb
```

## Reproducibility

- **Pinned environment:** `uv.lock` fixes every package version. Python is pinned to 3.12 in `.python-version`
  and `pyproject.toml`.
- **Seeds:** the notebook defines one constant, `SEED = 42`, in its first code cell. It seeds Python's `random`
  and NumPy, and is passed as `random_state` to every train/test split, cross-validation splitter, model and
  clustering algorithm. Bootstraps and permutations each get their own generator from `rng_for(task)`, derived from
  `SEED` and the task name, so a task's result does not depend on which cells ran before it. Changing `SEED` in that
  one place changes every random step.
- **Deterministic models:** LightGBM runs with `deterministic=True` and `force_row_wise=True`, so it gives the same
  result for any number of CPU threads. scikit-learn models are deterministic given `random_state`.
- **Verified:** two consecutive headless runs produce identical outputs in every cell. They also match the results
  committed in commit `f007d7e`. The only exception is the fit and inference **timings** reported in Section 3, which
  depend on the machine.
- **Raw data is read-only:** all cleaning is done in code inside the notebook, and the cleaned dataset is written to
  `data/processed/`.
- **Relative paths:** the notebook finds the project root by itself, so it runs from any working directory.

## Data

About 2,960 residential sales (2006–2010) with about 80 variables, provided "as is" by the client for academic use.
`data/Data_Dictionary.txt` describes each variable. The raw file contains data quality problems (duplicates,
corrupted target values, inconsistent category labels, impossible years). The notebook documents and treats them in
its data quality section.

## Key data decisions (Phase 1)

The full log is in `data/processed/data_quality_log.csv` and in Section 1 of the notebook.

- **Never invent a sale price:** 26 rows with a missing, negative or placeholder price are removed.
- **Repairs use evidence in the same row:** text formats, lot areas recorded in square meters, inconsistent category
  labels, impossible years, sentinel garage capacities (`Garage Cars = 99`), and unknown neighborhoods (recovered from the nearest parcel ID, 98.8% accurate) are fixed.
  Duplicate records of the same sale are collapsed to one.
- **Features that don't exist** (no pool, no garage, ...) are encoded as `"Absent"`, not imputed. Real gaps are imputed
  inside the model pipeline, which is fitted on training data only.
- **Analysis sample:** open-market sales (Normal and new homes, flagged `is_new_home`) up to 4,000 sqft of living area.
  Foreclosure-type, family, allocation and land-adjoining sales are excluded as non-market prices.
- **Lot Frontage** is dropped (`DROP_LOT_FRONTAGE = True`): confirmed via 5-fold training CV (0.02 pp MAPE difference), while avoiding non-random imputation on cul-de-sac lots.

## Use of AI tools

As the brief requires, the use of AI tools is declared in the notebook. Claude (Anthropic, via Claude Code) was used during initial Phase 0–1 setup, and Google Antigravity (Gemini) was used to extend the audit, complete Phases 2–7, verify reproducibility, and generate the executive presentation. All methodological choices and results were verified against executed outputs.
