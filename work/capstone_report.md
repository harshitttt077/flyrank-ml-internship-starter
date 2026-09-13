# Capstone Report — Lane 2: Refresh / Content Opportunity Scoring

- **Author:** Harshit Kudhial (`@harshitttt077`)
- **Lane:** Lane 2 — Refresh / Content Opportunity Scoring
- **Repo:** [https://github.com/harshitttt077/flyrank-ml-internship-starter](https://github.com/harshitttt077/flyrank-ml-internship-starter)
- **Date:** September 2026

---

## 0. Abstract

How can enterprise search editorial teams with strict monthly review capacities (e.g., 50 pages per month) prioritize which published assets to update among tens of thousands of degrading URLs? Using an anonymized multi-domain dataset of 30,000 published pages across 200 client websites from the FlyRank Search Intelligence Warehouse, we formulate content refresh prioritization as a Learning-to-Rank task evaluated by Precision@50 under client-holdout validation. While standard industry heuristics (e.g., refreshing stale pages with high historical traffic) achieve a Precision@50 of only 0.240—underperforming the 0.542 population base rate due to survivorship bias on mature evergreen assets—our leak-free Random Forest ranking engine achieves a Precision@50 of 0.680 (a **2.83x lift**, yielding +22 additional correctly prioritized declining assets per cycle). Target leakage is strictly prevented by excluding retrospective slope indicators (`trend_pct`, `trend_direction`), and cross-domain memorization is eliminated through an 80/20 grouped client split (27,675 train rows across 160 clients, 2,325 holdout rows across 40 unseen clients). These rankings directly drive an explainable editorial review queue categorized by diagnostic reason codes (`stale_visible_page`, `page_one_decay_risk`, `thin_visible_page`), providing decision-support intelligence for content marketing operations without making unsubstantiated causal claims.

---

## 1. Problem Framing

### The Decision Supported
Enterprise search and content marketing teams manage massive content libraries (typically 10,000 to 50,000 published assets per domain). Over time, search algorithms evolve, consumer query intent drifts, and competing domains publish fresher, more authoritative content. This leads to compounding traffic decay across established assets.

However, editorial human capacity is asymmetric: an editorial team can thoroughly audit, rewrite, expand, and re-optimize only **20 to 50 pages per month**. Content leaders face an urgent operational decision every month: *Which exact 50 pages out of 30,000 should editors spend their limited working hours updating?*

- **Unit of Analysis:** A single published content asset (`content_id`) within an enterprise client domain (`client_id`).
- **Output:** An ordered, scored priority review queue with assigned diagnostic reason codes and suggested editorial actions.
- **Action Taken:** Editorial teams audit, refresh facts, improve search intent alignment, update title tags, or expand thin sections.
- **Cost of a Wrong Call:** 
  - *False Positive:* Assigning an editor to update a stable or fluctuating asset wastes 6–8 hours of specialized labor ($400–$800 per page) with zero incremental organic lift.
  - *False Negative:* Missing a declining top-tier pillar page causes loss of Page 1 SERP real estate, directly impairing organic lead generation and revenue.

Data and ML help by identifying subtle non-linear multi-signal interactions (freshness, SERP positioning drift, CTR cliffs, engagement decay) that manual heuristics cannot detect.

---

## 2. Data Safety & Hygiene

### Data Used
We utilize the **FlyRank Search Intelligence Warehouse** anonymized 30,000-page cohort (`data/raw/content_refresh_anonymized.csv`), containing 44 metrics spanning trailing 90-day search impressions, clicks, sessions, average SERP position, CTR, content age, days since update, and engagement rates across 200 client domains.

### Columns Deliberately Excluded (and Why)
1. **Pseudonymous Identifiers (`client_id`, `content_id`):** Retained strictly for grouping and indexing. Excluded from features to prevent the model from memorizing specific client domains.
2. **Target-Derived Slope Fields (`trend_pct`, `trend_direction`):** Retained strictly to form the ground-truth label (`is_declining_label = (trend_direction == 'down')`). Strictly excluded from features because `trend_direction` is derived directly from trailing percentage slope; including them introduces catastrophic target leakage, yielding an artificial 1.000 precision that fails completely in production.
3. **Zero Client PII:** No URLs, domain names, customer brands, or raw search queries appear anywhere in the repository.

---

## 3. Baseline Heuristic

To establish a fair benchmark, we implemented the standard deterministic rule widely used by SEO practitioners:
$$\text{Baseline Rule} = (\text{days\_since\_last\_update} \ge 180) \land (\text{impressions\_90d} \ge \text{median})$$
Eligible candidates are scored by a composite weighting of visibility, freshness risk, position opportunity, and depth gap, and ranked descending.

### Baseline Performance on the Client-Holdout Set (2,325 Rows):
- **Precision@20:** 0.1500 (3 / 20 correct picks)
- **Precision@50:** 0.2400 (12 / 50 correct picks)
- **Precision@100:** 0.3600 (36 / 100 correct picks)
- **Holdout ROC-AUC:** 0.6269

*Why the baseline fails:* The hand-rule suffers from **survivorship bias**. Many high-impression pages older than 180 days are mature, high-authority evergreen pillars that remain highly stable without updates. Stale age alone is a poor indicator of active decay.

---

## 4. Model / Analysis

### Method & Feature Architecture
We trained an interpretable, non-linear **Random Forest Classifier** (`n_estimators=200`, `max_depth=10`, `min_samples_leaf=25`, `class_weight='balanced_subsample'`, `random_state=42`) alongside Decision Tree and Logistic Regression benchmarks.

- **Feature Count:** 52 engineered pre-decision features:
  - *Numeric Features:* `search_volume`, `competition`, `cpc`, `word_count`, `char_count`, `log_impressions_90d`, `log_clicks_90d`, `log_sessions_90d`, `log_ai_sessions_90d`, `days_with_impressions`, `days_with_sessions`, `content_age_days`, `days_since_last_update`, `ctr`, `avg_position`, `engagement_rate`, `scroll_rate`, `ai_traffic_pct`.
  - *One-Hot Categoricals:* `competition_level`, `content_type`, `main_intent`, `age_tier`, `freshness_tier`, `word_count_tier`, `impression_tier`, `position_tier`.
- **Target Definition:** $\text{is\_declining\_label} = \mathbb{I}(\text{trend\_direction} == \text{'down'})$.

---

## 5. Evaluation

### Split Strategy: Grouped Client Holdout
We partition the 200 client domains into an **80/20 grouped split** (160 training domains with 27,675 rows, 40 holdout domains with 2,325 rows). Test clients are completely unseen during training.

### Holdout Benchmark Results ($K = 50$):

| Metric | Population Base Rate | Hand-Rule Baseline | Decision Tree | Random Forest (Best) | Lift vs. Baseline |
|---|:---:|:---:|:---:|:---:|:---:|
| **Precision@20** | 0.542 | 0.150 | 0.500 | **0.700** | **4.67x** |
| **Precision@50** | 0.542 | 0.240 | 0.540 | **0.680** | **2.83x** |
| **Precision@100** | 0.542 | 0.360 | 0.530 | **0.700** | **1.94x** |
| **ROC-AUC** | 0.500 | 0.627 | 0.742 | **0.747** | +0.120 |
| **Average Precision** | 0.542 | 0.468 | 0.575 | **0.610** | +0.142 |

### Error Analysis & Interpretation
- The Random Forest achieves **0.680 Precision@50**, correctly identifying **34 out of 50 declining assets** (compared to only 12 for the baseline). This represents a **+22 page improvement** per monthly editorial cycle.
- The 16 false positives in the model's top 50 are not low-quality pages; they are "stable high-volume" pages with early leading indicators of engagement softening that have not yet crossed the -10% traffic threshold.

---

## 6. Interpretation

### Top Feature Importances (Gini):
1. `days_with_impressions` (16.0%): Consistency of search presence is the strongest signal separating stable assets from degrading ones.
2. `log_impressions_90d` (12.8%): Captures asset visibility and total market opportunity.
3. `avg_position` (10.8%): Movement into Page 2 striking distance (positions 11–25) strongly correlates with traffic drops.
4. `content_age_days` (9.5%): Age provides context when combined with interaction signals.
5. `word_count` & `char_count` (8.1%): Depth gap signal.
6. `ctr` & `scroll_rate` (6.4%): Intent mismatch leading indicators.

---

## 7. Recommendations & Action Playbook

### Diagnostic Reason Codes & Action Mapping
The model outputs a prioritized queue tagged with clear diagnostic explanations:
1. `stale_visible_page`: Trailing impressions $>5,000$ and $>180$ days since last update $\rightarrow$ **Action:** Comprehensive factual and structural refresh.
2. `page_one_decay_risk`: Positions 1–10 experiencing engagement drop $\rightarrow$ **Action:** Immediate protection refresh to safeguard rank.
3. `thin_visible_page`: High impressions but $<1,200$ words $\rightarrow$ **Action:** Expand content depth and add comprehensive sub-topics.
4. `low_ctr_visible_page`: High impressions with CTR $<0.5\%$ $\rightarrow$ **Action:** Optimize SERP title tag and meta description snippet.

---

## 8. Reproducibility

### Exact Commands to Replicate
```bash
# Clone the repository
git clone https://github.com/harshitttt077/flyrank-ml-internship-starter.git
cd flyrank-ml-internship-starter

# Set up Python 3.11/3.12 virtual environment
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Run the complete end-to-end pipeline
python scripts/run_all.py

# Or execute the capstone notebook
python scratch/execute_capstone_nb.py
```
- **Random Seeds:** 42 across all splits, permutations, and model estimators.
- **Environment:** Scikit-Learn 1.4+, Pandas 2.2+, NumPy 1.26+, Python 3.11/3.12.
- **Committed Metrics File:** `outputs/model_results.json` and `outputs/refresh_queue_sample.csv`.

---

## 9. Acknowledgments & Data Credit

Built on the **FlyRank ML Internship dataset** linking to [https://flyrank.ai](https://flyrank.ai). Sincere thanks to the FlyRank team for providing authentic enterprise search intelligence data.
