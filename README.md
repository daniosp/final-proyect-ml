# Clearview Properties — Residential Price Analysis

Final project for the *Aprendizaje Automático* (Machine Learning) course, MSc in Engineering Sciences, EAFIT (2026-2).

A real estate agency wants its brokers to use evidence from about 2,960 of its past home sales (2006–2010). The brief
(`task_info/Brief_Candidate_DataScientist.docx`) asks four questions, which the notebook answers:

| # | Question | How it is answered | Notebook |
|---|---|---|---|
| Q1 | What listing **price range** fits a home? | A blend of three tuned models, with listing ranges calibrated per price tier | Section 3 |
| Q2 | What **drives price**: what an owner can change, or structure and location? | Importance of groups of characteristics, plus location premiums | Section 4 |
| Q3 | Which **improvements** pay for themselves? | Simulated price uplift (= break-even cost), compared with published 2008 costs | Section 5 |
| Q4 | Are there **market segments** that behave differently? | Clustering on physical characteristics, then testing whether price drivers differ | Section 6 |

## Where to start

Open `main/clearview_analysis.ipynb`. The first cell has an **executive summary** (one answer per question, linked to
the evidence) and a table of contents. Each section opens with an **"In brief"** box and ends with a **"Decisions and
rationale"** box. Code cells are collapsed, so the text, tables and figures read as a report; expand a cell to see
its code. All numbers are printed by code, and the text refers to those outputs. Section 7 collects the answers,
the limitations, the next steps and a **model card** for operating the price model.

## How to run

You need [uv](https://docs.astral.sh/uv/getting-started/installation/); it installs Python 3.12 by itself.

```bash
git clone <repo-url>
cd proyecto-final-ml
uv sync --locked                 # exact package versions from uv.lock
uv run jupyter lab main/clearview_analysis.ipynb
```

In VS Code, open the notebook and choose the `.venv` interpreter as the kernel. To run everything without opening it
(about 13 minutes on a laptop, mostly model tuning and the Q3 refits):

```bash
uv run jupyter nbconvert --to notebook --execute --inplace main/clearview_analysis.ipynb
```

If uv reports `invalid peer certificate: UnknownIssuer` (common behind antivirus or corporate TLS inspection), add
`--system-certs` or set `UV_SYSTEM_CERTS=1`. To add a dependency, use `uv add <package>` and commit both
`pyproject.toml` and `uv.lock`.

## Reproducibility

- **Pinned environment:** `uv.lock` fixes every package; the first code cell stops if the kernel is not Python 3.12.
- **One seed:** `SEED = 42` drives every split, model and clustering. Each bootstrap or permutation gets its own
  random generator, derived from the seed and the task name, so results do not depend on which cells ran before.
- **Deterministic models:** LightGBM runs in deterministic mode, so results do not depend on the number of CPU threads.
  Only the fit and prediction timings in Section 3 vary between machines.
- **Raw data is never modified:** all cleaning happens in the notebook, and every issue is logged.

## Repository structure

```
data/
  Data.csv, Data_Dictionary.txt   raw sales and variable descriptions, as provided by the client
  processed/                      written by the notebook (git-ignored): data quality log, cleaned data,
                                  analysis sample, variable glossary
main/
  clearview_analysis.ipynb        the full analysis, Sections 0-7
  figures/                        charts exported by the notebook
  Clearview_Properties_Executive_Deck.pdf   client presentation, edited by the team
models/                           saved price model (git-ignored, regenerated on each run)
task_info/                        the brief
pyproject.toml, uv.lock           environment
```

## Main decisions

The reasons for each decision are in the notebook's decision boxes; these are the ones that shape the results.

- **Cleaning (Section 1):** sale prices are never repaired; invalid ones are removed. Other repairs use evidence from
  the same record. Empty values mean "feature absent" only where the data dictionary says so.
- **Scope:** open-market sales (normal and new homes) with up to 4,000 sqft of living area. Foreclosures, family sales
  and very large homes follow different rules.
- **No test-set leakage (Sections 2–3):** the 80/20 split comes first. Exploration, variable selection and tuning
  use the training data only, and the test set is used once.
- **Rules fixed before results:** the number of variables, whether to blend models, and the number of segments each
  follow a rule written before the results were seen.
- **Price ranges (Q1):** model errors on past sales set the width of the 80% listing range and the 50%
  negotiation band, separately for each price tier.
- **Drivers (Q2):** association, not causation. Characteristics are grouped by whether an owner can change them.
- **Improvements (Q3):** the break-even cost is the main result. Published costs (*Remodeling* magazine, Cost vs.
  Value 2008–09, West North Central region) are used for the four improvements they cover.
- **Segments (Q4):** clustering uses physical characteristics only. Price and location describe the segments
  afterwards.

## Outputs

- **Figures:** `main/figures/`, numbered by section.
- **Data files:** `data/processed/`. The data quality log records every issue, the rows affected, the action taken
  and why. The variable glossary records each variable's meaning and whether it was kept.
- **Model:** `models/clearview_price_model.joblib` holds the fitted models, blend weights and range calibration. The
  model card in Section 7 shows how to load it, and checks that it reproduces the notebook's valuations.

## Use of AI tools

Claude (Anthropic, through Claude Code) helped set up the environment and the data audit. Google Antigravity
(Gemini) helped write a first version of Sections 2–7 and of the presentation. Claude was then used to review the
notebook section by section and to improve its readability. The tools proposed code and alternatives, and the team
made the decisions. The full statement is in the notebook's first cell.
