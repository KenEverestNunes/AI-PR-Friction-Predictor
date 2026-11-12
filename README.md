# Predicting Friction in AI-Generated Pull Requests

> Early-warning classifier that flags AI-generated pull requests likely to trigger heavy code review (>=2 cycles) - before they waste engineering time.

- **80.7% recall** and **61.0% F1** on heavy-review PRs: catches 8 out of 10 problematic PRs before review begins
- **33,600 PRs** analyzed across 5 AI coding agents (OpenAI Codex, Copilot, Devin, Cursor, Claude Code)
- Solved a severe **14% / 86% class imbalance** - a naive majority-class baseline hits 86% accuracy with 0% recall

---

## Problem

AI coding agents generate PRs that frequently stall in multi-round code reviews, burning human reviewer bandwidth and slowing velocity. This project builds a pre-review classifier to flag high-friction PRs at submission time using metadata, code churn, and agent identity.

**Target definition:** `Heavy Review` = PR requiring >=2 review cycles. Single-cycle PRs reflect healthy human-in-the-loop collaboration; 2+ cycles signal a fundamental gap between AI output and maintainer expectations before velocity collapses at 3+ cycles.

---

## Data Pipeline

```
AIDev-pop Dataset (raw GitHub PR metadata, reviews, commit details)
  |- 5 AI agents x 13 file types (prs.csv, pr_reviews.json, pr_commit_details.json, ...)
        |
        v
  Feature Engineering (33,600 x 18 matrix)
  +-------------------------------------------------------------------+
  | Timeline:    hours_to_merge (created_at -> merged_at, 0 if open)  |
  | Code churn:  total_additions, total_deletions,                    |
  |              files_touched, total_changes                         |
  | Context:     pr_desc_length (word count of PR body)               |
  | Categorical: task_type (OHE: bug_fix, feature, docs, refactor,    |
  |              other), agent_identity (OHE across 5 AI agents)      |
  +-------------------------------------------------------------------+
        |
        v
  Class Imbalance Handling (86% Smooth vs. 14% Heavy)
  |- Evaluated: SMOTE oversampling (60% recall) & 0.30 threshold tuning (69% recall)
  |- Selected:  XGBoost scale_pos_weight (~6.40 ratio) for 6x minority-class penalty
        |
        v
  Model Selection (80/20 split, RandomizedSearchCV: 3-fold CV, 30 candidates, F1)
  |- Baseline: Random Forest (class_weight="balanced")
  |- Linear:   Logistic Regression (StandardScaler + class_weight="balanced")
  |- Champion: XGBoost (tuned, no PCA)
        |
        v
  Holdout Evaluation (n = 6,720 PRs: 5,780 Smooth / 940 Heavy)
```

**Engineering decision - no PCA:** Dimensionality reduction was intentionally skipped during hyperparameter tuning to preserve raw feature interpretability, keeping feature importance directly actionable for engineering teams.

---

## Benchmarks & Findings (Holdout Set, n = 6,720)

| Model | Accuracy | Precision (Class 1) | Recall (Class 1) | F1 (Class 1) |
|---|---|---|---|---|
| Random Forest (baseline) | 89.5% | 69.2% | 45.1% | 54.6% |
| Logistic Regression (linear) | 75.3% | 34.4% | 84.5% | 48.9% |
| **XGBoost (tuned champion)** | **85.6%** | **49.0%** | **80.7%** | **61.0%** |

*Baseline Random Forest ROC-AUC: 0.9168. Tuned XGBoost best parameters: `n_estimators=300`, `max_depth=9`, `learning_rate=0.01`, `subsample=0.7`, `colsample_bytree=0.8`, `scale_pos_weight=6.40`.*

**Confusion matrix (XGBoost champion):**

```
                 Pred: Smooth   Pred: Heavy
Actual: Smooth      4,970           810
Actual: Heavy         179           761   (80.7% recall)
```

### Key Takeaways

- **Code deletions outrank additions** as a friction predictor - removing or refactoring existing code triggers more human pushback than net-new additions.
- **Agent identity outweighs task type** - which AI tool generated the PR (notably OpenAI Codex) is a stronger signal than whether the PR is a bug fix or feature.
- **Precision/recall trade-off:** Random Forest misses over half of high-friction PRs (45.1% recall) and Logistic Regression causes alert fatigue (34.4% precision). Tuned XGBoost intercepts 8 of 10 high-friction PRs at 49.0% precision.

---

## Tech Stack

| Layer | Tools |
|---|---|
| Language | Python 3.11 |
| Data Processing | pandas, numpy, json |
| Modeling & Tuning | scikit-learn, XGBoost, imbalanced-learn |
| Visualization | matplotlib, seaborn |
| Environment | Google Colab / Jupyter |

---

## Quickstart

**Option A - Google Colab:**  
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/KenEverestNunes/AI-PR-Friction-Predictor/blob/main/Predicting_Friction_in_AI_Generated_Code.ipynb)

**Option B - Local:**
```bash
git clone https://github.com/KenEverestNunes/AI-PR-Friction-Predictor.git
cd AI-PR-Friction-Predictor
pip install -r requirements.txt
jupyter notebook Predicting_Friction_in_AI_Generated_Code.ipynb
```
*Note: Raw dataset files (`data/aidev_pop/`) are excluded via `.gitignore`. To re-run the pipeline locally or in Colab, download the `AIDev-pop` dataset from [Hugging Face](https://huggingface.co/datasets/hao-li/AIDev) into `data/aidev_pop/scope/` (or set `AIDEV_DATA_DIR`).*

---

## Repository Structure

```
|-- Predicting_Friction_in_AI_Generated_Code.ipynb  # End-to-end pipeline
|-- requirements.txt
|-- .gitignore
|-- figures/
|   |-- feature_importance.png
|-- presentation/
|   |-- Predicting-Heavy-Review-Cycles-in-AI-Generated-Pull-Requests.pptx
|   |-- Project-Report.pdf
|-- data/
    |-- README.md                                   # Dataset schema + upstream source
```

> **Dataset note:** Raw `AIDev-pop` files (both PR CSVs containing third-party scraped PR text and large `pr_commit_details.json` / `pr_timelines.json` tables up to 893 MB) are excluded via `.gitignore`. See [`data/README.md`](data/README.md) for the official dataset links.

---

## Dataset: AIDev-pop

Source: [hao-li/AIDev on Hugging Face](https://huggingface.co/datasets/hao-li/AIDev) ([Research Repo](https://github.com/SAILResearch/AI_Teammates_in_SE3)) - `AIDev-pop` subset covering repositories with >100 GitHub stars.

| AI Agent | Pull Requests |
|---|---|
| OpenAI Codex | 21,800 |
| GitHub Copilot | 4,971 |
| Devin | 4,829 |
| Cursor | 1,541 |
| Claude Code | 459 |
| **Total** | **33,600** |

*SE-611 Software Analytics - Drexel University*
