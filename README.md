# FDIC Bank Failure Early Warning — BFT65B

## Overview

An end-to-end data-engineering study of FDIC institution-level financial statements from 1984Q1 to 2026Q2. The project studies efficient data storage and processing (Q1), data quality and point-in-time correctness (Q2), and honest out-of-time bank-failure prediction (Q3).

**Public data source:** [FDIC BankFind Suite API](https://api.fdic.gov/banks/docs/). No API key is required. Financials (1,682,616 bank-quarters; 170 quarters; 24,064 CERTs; 47 selected fields), Failures (4,117 records), Institutions (27,835 records), and the supplementary History endpoint (506,824 events) were used in the original verified extraction. FDIC can retrospectively revise its API records; future reruns may differ.

## Repository contents

- `notebooks/` — nine final *source-code versions* for Parts 1–5; use the executed copies from Colab for final submission where available.
- `metadata/` — FDIC YAML field definitions; `FDIC_Data_Dictionary_Updated.xlsx` is a project-generated dictionary, not a bank-level dataset.
- `.gitignore` — excludes bank-level data, Parquet files, local caches and credentials.

**No bank-level datasets are distributed via GitHub.** Reproduce inputs using the API and the notebooks. Do not commit `data_raw/`, `data_storage/`, `data_processed/`, or full-bank CSV/Parquet files.

## Environment and setup

1. Use Google Colab with Python and internet access to the public FDIC API. Google Drive is used for persistent intermediates; PySpark runs as `local[2]`, not on a distributed cluster.
2. Create `MyDrive/FDIC_Bank_Failure_Project/`. Upload the four files in `metadata/` to `MyDrive/FDIC_Bank_Failure_Project/metadata/`.
3. Place the notebooks in Colab or Google Drive. For the submitted runs, notebooks use `/content/drive/MyDrive/FDIC_Bank_Failure_Project` as the default project root. Some support the `FDIC_PROJECT_ROOT` environment variable; Notebook 01 defaults directly to the Drive root.
4. Install exact requirements with `%pip install -r /content/requirements.txt` after uploading `requirements.txt` into the Colab runtime, **or** run the package-installation cells included in each notebook. The latter use version ranges, so record the concrete environment versions used in the final run.
5. If `requirements.txt` has not yet been generated, install all required libraries in Colab, run `scripts/generate_requirements.py`, and download the resulting pinned file. Do not use placeholder versions.

For the original study, the Part 3 benchmark reports Polars 1.35.2, pandas 2.2.3, PySpark 4.0.5; the Part 5 runtime records Python 3.13.18, pandas 2.2.3, NumPy 2.4.6, scikit-learn 1.6.1. These are **observed reference versions**, not a substitute for a complete `requirements.txt`.

## Run notebooks in the following order

| Step | Notebook | Purpose and important settings |
| --- | --- | --- |
| 1 | `01_FDIC_Data_Dictionary_Colab.ipynb` | Build the 47-field dictionary from the YAML metadata files; write `FDIC_Data_Dictionary_Updated.xlsx` to the Drive project root. |
| 2 | `02_FDIC_Acquisition_Storage_AUX_ID_REPAIR.ipynb` | Use `RUN_MODE='FULL'` (already set), `DOWNLOAD_HISTORY=False` and the post-download API audit to obtain and verify Financials, Failures and Institutions; create curated year-partitioned Parquet. |
| 3 | `03_FDIC_Data_Quality_and_Clean_Panel_FINAL.ipynb` | The consolidated Raw QA + Cleaning/03B notebook. Execute its Part A then Part B; produce `data_processed/clean_panel_v1`, QA reports and preliminary exit outcomes. |
| 4 | `03C_FDIC_History_Exit_Verification.ipynb` | Use `RUN_MODE='FULL'`. Download / verify History, improve exit classification, and write `data_processed/part2_history_exit_reference_v1/` for Part 4. |
| 5 | `04_FDIC_Scalable_Processing_Stage1_Pilot.ipynb` | Correctness pilot (2019–2020), Polars–pandas–Spark parity and plans. Requires the cleaned Part 2 panel. |
| 6 | `04_FDIC_Stage2_EdgeCases_and_Benchmarks_FIXED.ipynb` | First run `RUN_MODE='PILOT'` with `CONFIRM_FULL=False`; check PASS gates. Then change to `RUN_MODE='FULL'` **and** `CONFIRM_FULL=True` and rerun to get S25/S50/S75/S100 benchmarks. |
| 7 | `04C_FDIC_Part3_Final_Results_and_Charts.ipynb` | Read the Stage 2 **FULL** report ZIP and generate final figures. It does not re-benchmark; supply the expected ZIP from Step 6. |
| 8 | `05_FDIC_Part4_Features_Labels_Temporal_Split_FIXED.ipynb` | Confirm 4Q label/splits; generate features and train-only fitted transformations. `RUN_PREPROCESSOR=True` is set in this notebook; keep it after verifying development gates. |
| 9 | `06_FDIC_Part5_Modelling_Validation_Figures_Revised.ipynb` | Train/compare models using TRAIN/VALIDATION. Default `RUN_FINAL_TEST=False`, `ALLOW_TEST_REEVALUATION=False` to protect the already-evaluated Test period. |

### Output checks from the original verified study

| Stage | Expected result |
| --- | --- |
| Part 1 | 170/170 quarters, 1,682,616 Financials records, 8/8 acceptance gates passed |
| Part 2 | No source bank-quarter rows deleted; 14,346 non-assisted mergers verified under event-matching rules |
| Part 3 | Stage 1 7/7 acceptance and 75/75 parity; Stage 2 8/8 acceptance, 53/53 synthetic edge cases and 240/240 parity |
| Part 4 | 11/11 Stage 1 acceptance gates; TRAIN 1,084,227 / VAL 158,131 / TEST 266,527 eligible observations; 78 distinct failed banks on Test |
| Part 5 | 9/9 development gates; Validation HGB Core AP about 0.5004, selected threshold about 0.019782 |
| Locked Test | Single finalized evaluation: AP about 0.2749, ROC-AUC about 0.9436, and 68/78 distinct failed banks alerted |

## Temporal design and locked Test protocol

Features use a one-quarter accounting lag with exact-quarter matching. The prediction label represents failure within the following four calendar quarters. The split is chronological: TRAIN 1984Q1–2005Q4; embargo 2006; VALIDATION 2007Q1–2011Q4; embargo 2012; locked TEST 2013Q1–2025Q2. All learned preprocessing uses TRAIN only. Model and threshold are selected using development data only. The recorded alarm threshold is approximately 0.019782.

The original locked Test was evaluated **once**, after freezing the development protocol. Do not rerun Test on the original output directory. `RUN_FINAL_TEST=False` and `ALLOW_TEST_REEVALUATION=False` are deliberate submission defaults. A reviewer performing a *first-time independent reproduction* on a new clean output root can enable `RUN_FINAL_TEST=True` **only after reproducing and freezing development** and confirming there is no existing `FINAL_TEST_EVALUATION.json`. Never override the re-evaluation guard to improve results.

## Reproducibility limitations

- Financial statement `REPDTE` is a quarter-end date, not a guaranteed public release date; the one-quarter lag is a conservative assumption.
- FDIC records can be revised and API pagination may be affected if underlying data change during extraction.
- The timing benchmark was conducted in a single Google Colab environment on local `/content` copies, not a multi-node Spark cluster; measured runtimes depend on the environment.
- Some notebook source copies in this bundle do not contain saved execution outputs. For the final GitHub submission, download the **executed** Colab copies when available, preserving code, figures and relevant gate outputs. The summary report artifacts from the original study can be included separately; do not publish bank-level datasets.
- Full API acquisition and benchmark reproduction take substantially longer than reading the already-computed report artifacts. Numerical matching of timing results across hardware is not expected.

## References and AI disclosure

The main report supplies the full APA 7 references for FDIC, PyArrow, Polars, Apache Spark, scikit-learn, Collier et al. (2003), Cole & White (2012), Betz et al. (2014), and OpenAI (2026). ChatGPT was used to assist with code review, debugging, report organization and language editing. The author remains responsible for verifying data outputs and interpretations.
