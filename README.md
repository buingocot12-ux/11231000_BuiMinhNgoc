# FDIC Bank Failure Early-Warning Pipeline | BFT65B

**Student:** Bui Minh Ngoc  
**Student ID:** 11231000  
**Repository:** https://github.com/buingocot12-ux/11231000_BuiMinhNgoc

## 1. Project overview

This project develops and evaluates a reproducible, bank-quarter data pipeline for early warning of US bank failures, using public records from the [FDIC BankFind Suite API](https://api.fdic.gov/banks/docs/). It addresses three questions:

1. How should four decades of banking reports be stored and processed on a single machine, and when would a Spark cluster be justified?
2. What data-quality problems exist, and how can the pipeline handle them without look-ahead bias?
3. Can the processed data support an honestly validated, out-of-time bank-failure early-warning model?

The original verified dataset covers **1984Q1–2026Q2 (170 quarters)**, with **1,682,616 Financials bank-quarter records**, **24,064 institution certificate numbers**, and **47 selected Financials fields**. Additional records comprise **4,117 Failures**, **27,835 Institutions**, and **506,824 History events**. The last endpoint was used for institutional-exit verification. These are counts from the original study, not a guarantee that a fresh FDIC API download will return exactly the same snapshot.

**No raw or processed bank-level datasets are committed to this repository.** The notebooks obtain input data from the public API; the repository retains selected summary reports and figures as evidence of the original run.

## 2. Repository structure

Paths below reflect the **actual directory names in this repository**, including capitalization and spaces:

```text
11231000_BuiMinhNgoc/
├── README.md
├── requirements.txt
├── .gitignore
├── Notebook/                     # Nine notebooks for Parts 1–5
├── metadata/                     # FDIC YAML definitions and project data dictionary
└── Report/
    ├── report/                   # Aggregate result files for Parts 1–5
    │   ├── part 1/
    │   ├── part 2/
    │   ├── part3/
    │   ├── part4/
    │   └── part5/
    └── figure_part 5/            # Three Part 5 figures
```

The `Notebook/` directory stores executable source code. `metadata/` contains the four FDIC YAML metadata files and a project-generated XLSX data dictionary, not institutional financial observations. `Report/report/` contains small CSV/JSON/TXT audit, benchmark, and evaluation summaries. `Report/figure_part 5/` contains the final model-evaluation charts.

## 3. Environment and installation

The original analyses were conducted in **Google Colab** with persistent intermediate files in **Google Drive**. Part 3 ran Spark in **`local[2]`** mode on one machine, not a distributed cluster. The Part 5 runtime report recorded Python **3.13.18**, pandas **2.2.3**, NumPy **2.4.6**, and scikit-learn **1.6.1**; see [runtime_versions.json](Report/report/part5/runtime_versions.json). The benchmark reports recorded Polars **1.35.2**, pandas **2.2.3**, and PySpark **4.0.5**.

1. Open Google Colab and connect a runtime. The pinned package versions are listed in [`requirements.txt`](requirements.txt). Upload that file to `/content/requirements.txt` in Colab.
2. In a **new Colab code cell**, install the dependencies:

   ```python
   %pip install -r /content/requirements.txt
   ```

3. Create the folder `MyDrive/FDIC_Bank_Failure_Project/` in your Google Drive, then create its `metadata/` subfolder.
4. Copy the following four files from this repository's [`metadata/`](metadata/) directory into `MyDrive/FDIC_Bank_Failure_Project/metadata/`: `risview_properties.yaml`, `failure_properties.yaml`, `institution_properties.yaml`, and `history_properties.yaml`.
5. Open the notebooks in Colab in the order below. Most notebooks use the project root `/content/drive/MyDrive/FDIC_Bank_Failure_Project`; several also permit overriding it through `FDIC_PROJECT_ROOT`.

**Important:** Runtime packages, Java/PySpark compatibility, API response changes, and Colab resources can affect independent reproduction. The pinned requirements describe the submitted environment but have not been verified in this repository through an automated clean-environment CI run.

## 4. Notebook execution order

Open the linked notebook, inspect its configuration cell, and run its cells in the specified order. The source filenames include `(1)` or `(2)` only where those suffixes are present in the GitHub repository.

| Step | Notebook | Purpose / required action |
|---|---|---|
| 1 | [01 — FDIC Data Dictionary](Notebook/01_FDIC_Data_Dictionary_Colab.ipynb) | Read the four YAML files and build the project data dictionary. |
| 2 | [02 — Acquisition and Storage (ID repair)](Notebook/02_FDIC_Acquisition_Storage_AUX_ID_REPAIR.ipynb) | Set `RUN_MODE='FULL'` for the complete study; acquire Financials, Failures, and Institutions and verify API IDs and counts. The original configuration has `DOWNLOAD_HISTORY=False`; History is handled in Step 4. |
| 3 | [03 — Raw QA and Clean Panel (FINAL)](Notebook/03_FDIC_Data_Quality_and_Clean_Panel_FINAL.ipynb) | Run **Part A (raw QA), then Part B (clean panel and initial exit references)**. Requires Step 2's curated Financials. |
| 4 | [03C — History Exit Verification](Notebook/03C_FDIC_History_Exit_Verification.ipynb) | Keep `RUN_MODE='FULL'`; download History events and verify mergers/non-failure exits. Requires Step 3's outcomes. |
| 5 | [04 — Part 3 Stage 1 Pilot](Notebook/04_FDIC_Scalable_Processing_Stage1_Pilot%20%281%29.ipynb) | Check cross-engine correctness using the pilot sample. Requires both Step 2 Financials and Step 3 clean-panel files. |
| 6 | [04 — Part 3 Stage 2 Benchmarks (FIXED)](Notebook/04_FDIC_Stage2_EdgeCases_and_Benchmarks_FIXED%20%281%29.ipynb) | **Pilot first:** `RUN_MODE='PILOT'` and `CONFIRM_FULL=False`. After pilot gates pass, run the full benchmark with `RUN_MODE='FULL'` and `CONFIRM_FULL=True`. The submitted notebook is configured for the full run. |
| 7 | [04C — Part 3 Results and Charts](Notebook/04C_FDIC_Part3_Final_Results_and_Charts%20%281%29.ipynb) | The plotting notebook reads `FDIC_Part3_Stage2_Benchmark_Reports.zip` created by Step 6. It looks for that ZIP in `/content/` or prompts an upload; it does **not** rerun the benchmark. |
| 8 | [05 — Features, Labels and Temporal Splits (FIXED)](Notebook/05_FDIC_Part4_Features_Labels_Temporal_Split_FIXED%20%282%29.ipynb) | Requires Step 3 clean panel and event alignment plus Step 4 revised History outcomes. Builds 26 candidate features, target labels, chronological splits, and train-only preprocessing (`RUN_PREPROCESSOR=True`). |
| 9 | [06 — Modelling and Validation (REVISED)](Notebook/06_FDIC_Part5_Modelling_Validation_Figures_Revised%20%281%29.ipynb) | Compare models using TRAIN and VALIDATION, select the alarm threshold, and inspect locked-Test artifacts. **Keep `RUN_FINAL_TEST=False` and `ALLOW_TEST_REEVALUATION=False` for the existing study.** |

The end-to-end inputs and outputs are persisted under the Drive project folder, not under GitHub. A new run must acquire its own FDIC data and rebuild all intermediate datasets; the summary reports checked into GitHub are **not sufficient as model inputs**.

## 5. Methods: target, time split and leakage controls

The target `fail_within_4q` flags a confirmed FDIC bank failure within the next four calendar quarters. Predictors are joined to **exactly the preceding reporting quarter (`q−1`)**; missing preceding-quarter records are not filled by carrying forward older financial statements. Failure dates, mergers, and other retrospective outcomes are used for label verification, **not as model predictors**.

| Period | Quarters | Eligible bank-quarter rows | Positive quarters | Distinct failed banks |
|---|---|---:|---:|---:|
| Training | 1984Q1–2005Q4 | 1,084,227 | 8,278 | 2,191 |
| Embargo | 2006 | — | — | — |
| Validation | 2007Q1–2011Q4 | 158,131 | 1,791 | 465 |
| Embargo | 2012 | — | — | — |
| Locked Test | 2013Q1–2025Q2 | 266,527 | 254 | 78 |

The four-quarter embargo separates the development and final evaluation periods. Learned imputation, missingness indicators, and scaling are fitted using **Training only**. The true historical availability dates of financial statements are not available; a one-quarter reporting lag is a conservative assumption, not proof of public availability by that date.

## 6. Direct links to verification reports

The following **aggregate** outputs document the original completed run. They are included for auditing and are not substitutes for executing the notebooks:

| Part | Original-run evidence | What it verifies |
|---|---|---|
| 1 | [Acquisition gates](Report/report/part%201/acceptance_gates.csv) · [Source completeness](Report/report/part%201/source_completeness.csv) · [Storage benchmarks](Report/report/part%201/storage_benchmark.csv) | 170 reporting quarters, 1,682,616 Financials rows, 8/8 acceptance gates; Parquet vs CSV |
| 2 | [Raw QA summary](Report/report/part%202/raw_qa/audit_summary.json) · [Cleaning acceptance](Report/report/part%202/cleaning_v1/part2_clean_panel_acceptance.csv) · [History summary](Report/report/part%202/history_exit_v1/history_exit_summary.json) · [Exit transition table](Report/report/part%202/history_exit_v1/exit_class_transition_matrix.csv) | Data quality, original-row preservation, and 14,346 verified non-assisted mergers |
| 3 | [Pilot gates](Report/report/part3/pilot/stage1_acceptance_gates.csv) · [Full benchmark gates](Report/report/part3/full_benchmark/stage2_acceptance_gates.csv) · [Cross-engine parity](Report/report/part3/full_benchmark/stage2_parity_results.csv) · [Runtime and memory](Report/report/part3/full_benchmark/stage2_benchmark_summary.csv) | 7/7 pilot gates; 8/8 full gates, 240/240 parity checks; W1–W3 runtime and memory |
| 4 | [Feature catalog](Report/report/part4/candidate_feature_catalog.csv) · [Temporal split counts](Report/report/part4/split_counts_and_failures.csv) · [Leakage gates](Report/report/part4/stage1_acceptance_gates.csv) · [Preprocessing checks](Report/report/part4/stage2_preprocessing_validation.csv) | 26 candidate features, one-quarter lag, chronological splits, train-only preprocessing |
| 5 | [Validation comparison](Report/report/part5/validation_model_comparison.csv) · [Development gates](Report/report/part5/development_acceptance_gates.csv) · [Frozen protocol](Report/report/part5/locked_development_protocol.json) · [Locked Test evaluation](Report/report/part5/FINAL_TEST_EVALUATION.json) · [Test threshold metrics](Report/report/part5/FINAL_TEST_Threshold_Metrics.csv) | Model selection, frozen threshold, one-time out-of-time evaluation, confusion matrix |

### Final evaluation figures

- [Figure 1 — Validation precision–recall](Report/figure_part%205/Figure_1_Validation_PR.png)
- [Figure 2 — Validation cost-based threshold](Report/figure_part%205/Figure_2_Validation_Threshold_Cost.png)
- [Figure 3 — Locked Test precision–recall](Report/figure_part%205/Figure_3_FINAL_TEST_PR.png)

### Reference results

The original Stage 2 benchmark passed **8/8 acceptance gates**, **53/53 synthetic edge-case tests**, and **240/240 cross-engine comparisons**. On the 1,682,616-row W3 end-to-end transformation workload, the recorded median run times were **1.460 s (Polars)**, **4.651 s (pandas)**, and **9.496 s (Spark local)**. Timings are hardware- and environment-dependent.

The selected **HGB Core** model achieved **Validation AP 0.5004**. The development-selected alarm threshold was approximately **0.019782**. The **single locked Test** achieved **AP 0.2749**, **ROC-AUC 0.9436**, and alerted **68/78 distinct failed banks (87.18%)**, with **1,205 false-positive bank-quarter alerts**. See the linked evaluation JSON for exact values and the locked protocol's SHA-256 reference.

## 7. Locked Test handling and independent reproduction

The original Test was used **once**, after model and threshold selection had been frozen using the development data. On the submitted notebook, `RUN_FINAL_TEST=False` and `ALLOW_TEST_REEVALUATION=False` prevent accidental repeated evaluation. **Do not rerun or unlock the original Test simply to obtain different metrics.**

For an **independent first-time reproduction**, use a new output root and rerun acquisition, QA, History verification, feature construction, and development modelling in the specified order. After confirming the development checks and freezing the development protocol, a reviewer may enable the first Test execution in that **new, clean run**. Do not reuse or overwrite the original `FINAL_TEST_EVALUATION.json` or bypass the duplicate-evaluation guard.

## 8. Known limitations

- FDIC may revise historical records; future API extracts may not match the original snapshot exactly.
- `REPDTE` marks quarter-end, not the confirmed public reporting date; actual information availability is uncertain.
- Unclear mergers and institutional exits were not automatically labeled as failures or healthy continuations.
- The reported Spark experiment used local execution rather than a multi-node cluster.
- Benchmark timing and memory figures depend on the Colab hardware and runtime environment.
- One notebook may be a source-code copy without saved outputs; aggregate evidence is supplied under `Report/report/`. Preserve executed notebook outputs when submitting the final version.
- There is no automated continuous-integration demonstration of a fresh end-to-end run in this repository.

## 9. Data access, references and AI disclosure

**Data provider:** Federal Deposit Insurance Corporation, [BankFind Suite API](https://api.fdic.gov/banks/docs/). Data download and verification methods are documented in the acquisition notebook and Part 1's [acquisition summary](Report/report/part%201/acquisition_summary.json). The verified snapshot timestamp, not necessarily the initial download start, is reported in the final assignment report.

**Methods and software documentation:** [PyArrow](https://arrow.apache.org/docs/python/index.html), [Polars Lazy optimizations](https://docs.pola.rs/user-guide/lazy/optimizations/), [Apache Spark SQL performance tuning](https://spark.apache.org/docs/latest/sql-performance-tuning.html), and [scikit-learn model evaluation](https://scikit-learn.org/stable/modules/model_evaluation.html).

**Bank-risk research:** Collier et al. (2003), *The SCOR system of off-site monitoring*; Cole and White (2012), *Déjà vu all over again*; Betz et al. (2014), *Predicting distress in European banks*. Complete APA 7 entries are provided in the submitted report.

**AI use disclosure:** ChatGPT (OpenAI) assisted with code review, debugging, writing organization, and language editing. Data extraction decisions, verification of reported metrics, modelling interpretations, and final submission remain the author's responsibility.
