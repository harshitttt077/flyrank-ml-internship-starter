# The Prompt Ladder (Week 2)
**Track**: General AI Fluency | **Phase**: Foundations | **Author**: Harshit Kudhial (`harshitttt077`)  
**Date**: September 2026 | **Deliverable**: 6-Stage Prompt Progression, Side-by-Side Output Notes & Reusable Master Prompt

---

## Overview: The Discipline of Single-Layer Iteration

The gap between a lazy prompt and an engineered prompt is the cheapest performance upgrade in AI. Most practitioners fail to unlock this because they bundle five changes simultaneously and cannot attribute what caused the output to shift.

This document records an authentic prompt ladder from the **Machine Learning / Search Analytics track**: engineering a prompt to prioritize decaying web assets for editorial refresh. Across six runs (a weak baseline plus five single-layer iterations), exactly one named ingredient is added per version, and each output is evaluated on observable differences.

---

## The 6-Stage Prompt Ladder

```text
[ RUN 0: WEAK BASELINE ] ──> Vague query; hallucinates schema, gives generic SEO blog advice.
         │
         ▼  (+ Layer 1: Real Context & Data Schema)
[ RUN 1: GROUNDED SCHEMA ] ──> Real columns used; but outputs unranked 16,000-row binary dump.
         │
         ▼  (+ Layer 2: Clear Operational Goal & Capacity Limit)
[ RUN 2: THE LEAKAGE TRAP ] ──> [HONEST FAILURE] Artificially perfect 1.000 precision; leaked label!
         │
         ▼  (+ Layer 3: Negative Constraints & Anti-Leakage Guard)
[ RUN 3: HONEST VALIDATION ] ──> Leaky fields banned; client-holdout split enforced; realistic 0.68 lift.
         │
         ▼  (+ Layer 4: Specified Output Format & Reason Codes)
[ RUN 4: STRUCTURED OUTPUT ] ──> Markdown review queue with explicit attribution & comparison table.
         │
         ▼  (+ Layer 5: Verification Requirements & Defensive Assertions)
[ RUN 5: PRODUCTION RUNNER ] ──> Self-verifying code with automated leakage & split integrity assertions.
```

---

### Run 0: The Embarrassing Baseline

#### The Prompt:
```text
Help me find which pages to refresh in my SEO data.
```

#### Output Excerpt:
> *"To find which pages to refresh in your SEO data, you should look at Google Analytics and Google Search Console. Export your data to CSV and write a Python script:
> ```python
> import pandas as pd
> df = pd.read_csv("google_analytics_data.csv")
> # Find pages where bounce rate is high and traffic dropped
> pages_to_refresh = df[(df["bounce_rate"] > 0.70) & (df["pageviews_drop"] > 0.30)]
> print("Refresh these pages:", pages_to_refresh["url"])
> ```
> Also make sure to update your meta descriptions, add new keywords, build more backlinks, and check your Core Web Vitals using PageSpeed Insights."*

#### Four Evaluation Notes:
1. **What changed in prompt**: Baseline run (no changes).
2. **What actually improved in output**: Baseline reference point.
3. **What still failed**: It hallucinated hypothetical columns (`bounce_rate`, `pageviews_drop`, `url`), assumed a standard Google Analytics export, dumped generic blogging advice, and failed to understand the actual schema or editorial prioritization.
4. **What to try next**: Add **Real Context** by injecting the actual 44-column FlyRank dataset schema and file location.

---

### Run 1: Adding Real Context & Data Schema

#### The Single Change:
Added **Real Context & Schema**: Provided the exact CSV path (`data/raw/content_refresh_anonymized.csv`) and listed actual pre-decision columns (`content_id`, `client_id`, `impressions_90d`, `ctr`, `avg_position`, `days_since_last_update`, `word_count`, `trend_direction`).

#### The Prompt:
```text
Help me find which pages to refresh in my SEO data.
Context: I have a dataset at `data/raw/content_refresh_anonymized.csv` with 30,000 rows. The columns include: `client_id`, `content_id`, `content_age_days`, `days_since_last_update`, `impressions_90d`, `ctr`, `avg_position`, `word_count`, and `trend_direction` ('up', 'down', 'stable', 'new', 'flat').
```

#### Output Excerpt:
> *"Here is a Python script using your specific columns:
> ```python
> import pandas as pd
> df = pd.read_csv("data/raw/content_refresh_anonymized.csv")
> # Filter for pages in decline that have significant search volume
> refresh_candidates = df[(df["trend_direction"] == "down") & (df["impressions_90d"] >= 100)]
> print(f"Found {len(refresh_candidates)} pages to refresh.")
> refresh_candidates.to_csv("refresh_pages.csv", index=False)
> ```
> This filters your 30,000 pages for any content where `trend_direction` is 'down' and 90-day impressions exceed 100."*

#### Four Evaluation Notes:
1. **What changed in prompt**: Added **Real Context & Data Schema** (exact file path and real column names).
2. **What actually improved in output**: The code completely stopped hallucinating non-existent columns (`bounce_rate`, `pageviews`); it correctly ingested the local CSV and used authentic field names (`trend_direction`, `impressions_90d`).
3. **What still failed**: It outputted an unranked binary filter that dumped **16,180 rows** into a CSV. It had zero concept of human editorial capacity (an editorial team cannot review 16,000 pages).
4. **What to try next**: Add a **Clear Operational Goal** specifying an ordered top-50 priority queue evaluated by Precision@50.

---

### Run 2: Adding Clear Operational Goal & Capacity Constraint (The "Made it Worse" Moment)

#### The Single Change:
Added **Clear Goal & Capacity Constraint**: Instructed the AI to build a machine learning model to output an ordered priority queue of exactly the top 50 pages for monthly editorial review, evaluated by Precision@50 against the baseline.

#### The Prompt:
```text
Help me find which pages to refresh in my SEO data.
Context: I have a dataset at `data/raw/content_refresh_anonymized.csv` with 30,000 rows. The columns include: `client_id`, `content_id`, `content_age_days`, `days_since_last_update`, `impressions_90d`, `ctr`, `avg_position`, `word_count`, `trend_pct`, and `trend_direction`.
Goal: An editorial team can only review 50 pages per month. Train a machine learning model to output a prioritized ranking of the top 50 pages most in need of refresh, maximizing Precision@50 on detecting whether a page's trend is 'down'.
```

#### Output Excerpt:
> *"Here is an ML model optimized to maximize Precision@50 for your top 50 review queue:
> ```python
> from sklearn.ensemble import RandomForestClassifier
> import pandas as pd, numpy as np
>
> df = pd.read_csv("data/raw/content_refresh_anonymized.csv")
> df["target"] = (df["trend_direction"] == "down").astype(int)
> features = ["content_age_days", "days_since_last_update", "impressions_90d", "ctr", "avg_position", "word_count", "trend_pct"]
> X = df[features].fillna(0)
> y = df["target"]
>
> model = RandomForestClassifier(n_estimators=100, random_state=42)
> model.fit(X, y)
> df["score"] = model.predict_proba(X)[:, 1]
> top_50 = df.sort_values("score", ascending=False).head(50)
> precision_50 = top_50["target"].mean()
> print(f"Precision@50: {precision_50:.4f}") # Outputs 1.0000!
> ```
> Your model achieves a perfect Precision@50 of 1.0000! All 50 pages are guaranteed declining targets."*

#### Four Evaluation Notes:
1. **What changed in prompt**: Added **Clear Goal & Capacity Constraint** (top-50 queue, Precision@50 metric).
2. **What actually improved in output**: The output shifted from an unmanageable dump of 16,000 rows to a focused, scored top-50 priority queue.
3. **What still failed (THE "MADE IT WORSE" FAILURE)**: **Catastrophic Target Leakage!** Desperate to maximize Precision@50, the AI eagerly included `trend_pct` in the feature matrix `X`. Because `trend_direction == 'down'` is derived directly from `trend_pct < -0.10`, the model simply memorized the label. The "perfect" 1.0000 score is an artificial illusion that completely destroys the real-world utility of the model. Furthermore, it evaluated in-sample on random rows.
4. **What to try next**: Add strict **Negative Constraints & Anti-Leakage Rules** forbidding label-derived features and enforcing client-holdout validation.

---

### Run 3: Adding Negative Constraints & Anti-Leakage Guard

#### The Single Change:
Added **Negative Constraints & Anti-Leakage Rules**: Explicitly banned `trend_pct` and `trend_direction` from features, and mandated a client-holdout split (`client_id` grouped split) so test clients are never seen in training.

#### The Prompt:
```text
Help me find which pages to refresh in my SEO data.
Context: I have a dataset at `data/raw/content_refresh_anonymized.csv` with 30,000 rows. The columns include: `client_id`, `content_id`, `content_age_days`, `days_since_last_update`, `impressions_90d`, `ctr`, `avg_position`, `word_count`, `trend_pct`, and `trend_direction`.
Goal: An editorial team can only review 50 pages per month. Train a machine learning model to output a prioritized ranking of the top 50 pages most in need of refresh, maximizing Precision@50 on detecting whether a page's trend is 'down'.
Constraints:
1. STRICT LEAKAGE BAN: Never use `trend_pct` or `trend_direction` as features—they encode the label. Only use pre-decision features: `content_age_days`, `days_since_last_update`, `impressions_90d`, `ctr`, `avg_position`, `word_count`.
2. VALIDATION RULE: Do not evaluate in-sample. Hold out 20% of `client_id`s so no client's pages appear in both train and test sets.
```

#### Output Excerpt:
> *"Here is the honest, leak-free ranking pipeline using client-holdout validation:
> ```python
> import pandas as pd, numpy as np
> from sklearn.ensemble import RandomForestClassifier
>
> df = pd.read_csv("data/raw/content_refresh_anonymized.csv")
> y = (df["trend_direction"] == "down").astype(int)
> features = ["content_age_days", "days_since_last_update", "impressions_90d", "ctr", "avg_position", "word_count"]
> X = df[features].fillna(0)
>
> # Client-holdout split
> clients = df["client_id"].unique()
> np.random.seed(42)
> test_clients = set(np.random.choice(clients, size=int(len(clients) * 0.2), replace=False))
> test_mask = df["client_id"].isin(test_clients)
>
> model = RandomForestClassifier(n_estimators=100, max_depth=6, random_state=42)
> model.fit(X[~test_mask], y[~test_mask])
> test_scores = model.predict_proba(X[test_mask])[:, 1]
>
> top_50_idx = np.argsort(-test_scores)[:50]
> holdout_p50 = y[test_mask].iloc[top_50_idx].mean()
> print(f"Honest Holdout Precision@50: {holdout_p50:.3f}") # Outputs 0.680
> ```"*

#### Four Evaluation Notes:
1. **What changed in prompt**: Added **Negative Constraints & Anti-Leakage Rules** (banning label fields and enforcing grouped client splits).
2. **What actually improved in output**: The artificial 1.0000 leakage collapsed to a real, honest **0.680 holdout Precision@50**; the split cleanly isolated clients, preventing memorization of domain authority.
3. **What still failed**: The output ended with a raw terminal print of numbers. It provided zero human-interpretable reason codes explaining *why* a page was ranked, and lacked a structured deliverable for the content team.
4. **What to try next**: Add a **Specified Output Format** requiring a structured Markdown table with explicit reason codes and baseline comparison.

---

### Run 4: Adding Specified Output Format & Reason Codes

#### The Single Change:
Added **Specified Output Format & Reason Codes**: Required outputting an executive Markdown summary comparing Baseline vs. Model Precision@50, and exporting the top 10 preview rows containing `rank`, `content_id`, `priority_score`, and human-readable `reason_codes` (e.g. `stale_high_visibility`, `ctr_cliff`).

#### The Prompt:
```text
[Prompt from Run 3]
Output Format:
1. Print a Markdown comparison table showing: Metric, Hand-Rule Baseline (`stale * visible`), Random Forest Model, and Lift Ratio.
2. Output the top 10 rows of the queue as a Markdown table with columns: `rank`, `content_id`, `priority_score`, and `primary_reason_code` (rule explaining the flag, such as 'high_impressions_low_ctr' or 'stale_pillar_content').
```

#### Output Excerpt:
> *"### Model Performance Comparison
> | Metric | Hand-Rule Baseline | Random Forest Model | Lift vs. Baseline |
> |---|:---:|:---:|:---:|
> | **Holdout Precision@50** | 0.240 | 0.680 | **2.83x** |
> | **Correct Picks in Top 50** | ~12 / 50 | ~34 / 50 | +22 pages |
>
> ### Top Priority Queue Sample (First 5 Rows)
> | Rank | Content ID | Priority Score | Primary Reason Code |
> |:---:|---|:---:|---|
> | 1 | `content_304f4823` | 0.942 | `stale_high_visibility_pillar` |
> | 2 | `content_a1fb4e70` | 0.918 | `high_impressions_ctr_cliff` |
> | 3 | `content_9aa793d4` | 0.895 | `striking_distance_position_decay` |
> | 4 | `content_5c68b731` | 0.881 | `stale_high_visibility_pillar` |
> | 5 | `content_2e147b90` | 0.874 | `thin_content_high_volume` |"*

#### Four Evaluation Notes:
1. **What changed in prompt**: Added **Specified Output Format & Reason Codes** (Markdown benchmark table + attributed queue columns).
2. **What actually improved in output**: The output stopped dumping raw arrays; it produced a publication-grade executive summary table and an actionable review queue with interpretable reason codes that an editorial team can immediately execute against.
3. **What still failed**: The code lacks defensive verification. If a row has null features or the input dataset shifts, the script silently runs or crashes without asserting data integrity.
4. **What to try next**: Add **Verification Requirements & Defensive Assertions** to ensure data pipeline integrity before emitting results.

---

### Run 5: Adding Verification Requirements & Defensive Assertions (Final Version)

#### The Single Change:
Added **Verification Requirements & Defensive Assertions**: Mandated automated assertions checking that input row count matches output row count, verifying zero NaN values in scores, asserting that holdout clients do not overlap with training clients, and asserting a minimum 2.0x lift over baseline.

#### The Prompt:
```text
[Prompt from Run 4]
Verification Requirements:
Include explicit Python assertions in the code to guarantee integrity:
- Assert that no `client_id` exists in both training and holdout sets (`assert len(set(train_clients).intersection(test_clients)) == 0`).
- Assert that candidate features contain zero target-derived columns (`assert 'trend_pct' not in features`).
- Assert that the output queue contains exactly 50 rows with zero null scores (`assert len(top_50) == 50 and top_50['priority_score'].isna().sum() == 0`).
- Assert that model Precision@50 achieves at least a 2.0x lift over the baseline (`assert model_p50 / baseline_p50 >= 2.0`).
If any assertion fails, halt with an informative error message.
```

#### Output Excerpt:
> *"```python
> # === Verification Pipeline Checks ===
> assert len(set(train_clients).intersection(test_clients)) == 0, "Split Contamination Error!"
> assert "trend_pct" not in features and "trend_direction" not in features, "Target Leakage Error!"
> assert len(top_50) == 50, "Queue Capacity Mismatch!"
> assert top_50["priority_score"].isna().sum() == 0, "Null Score Detected!"
> assert rf_p50 / base_p50 >= 2.0, f"Model underperformed threshold: {rf_p50 / base_p50:.2f}x"
> print("All 4 verification checks passed: Zero leakage, clean client split, 2.83x lift confirmed.")
> ```
> The script executes cleanly, validates all constraints defensively, and outputs the certified executive tables."*

#### Four Evaluation Notes:
1. **What changed in prompt**: Added **Verification Requirements & Defensive Assertions**.
2. **What actually improved in output**: The code graduated from an experimental research script to an enterprise-grade, self-auditing data pipeline; it defensively proves its own split integrity and guarantees absence of leakage before producing results.
3. **What still failed**: Nothing within the defined operational scope. The output is robust, attributed, leak-proof, and reproducible.
4. **What to try next**: Package this into a parameterized master prompt reusable by any engineer across the internship track.

---

## 3. The Final Reusable Master Prompt

Any engineer or intern on the ML/SEO track can copy-paste this prompt into Claude or ChatGPT to get a production-ready ranking pipeline:

```text
You are an expert Machine Learning Systems Engineer specializing in search performance analytics and Learning-to-Rank pipelines.

[CONTEXT & DATA]
I have an anonymized tabular dataset at `{DATASET_PATH}` with `{ROW_COUNT}` rows.
Each row represents one content asset (`content_id`) belonging to a client domain (`client_id`).
Pre-decision observable features: `content_age_days`, `days_since_last_update`, `impressions_90d`, `ctr`, `avg_position`, `word_count`, `engagement_rate`, `position_tier`.
Outcome proxy column: `trend_direction` ('up', 'down', 'stable', 'new', 'flat'). Target is `is_declining = (trend_direction == 'down')`.

[OPERATIONAL GOAL]
An editorial team has a capacity of exactly {CAPACITY_K} pages per month.
Train an explainable ranking model (Random Forest / Decision Tree) to output an ordered review queue of the top {CAPACITY_K} pages most in need of refresh.
Evaluate performance using Precision@{CAPACITY_K} on unseen clients, benchmarked against a hand-written baseline heuristic (`stale * visible`).

[STRICT CONSTRAINTS]
1. ZERO LEAKAGE: Never use `trend_pct` or `trend_direction` as features. They mathematically encode the label.
2. GROUPED CLIENT SPLIT: Hold out 20% of `client_id`s. No client may have pages in both training and test sets.
3. NO PII: Never output raw URLs or client names.

[VERIFICATION REQUIREMENTS]
Include explicit Python assertions:
- Assert zero client overlap between train and test sets.
- Assert candidate feature matrix excludes all label-derived columns.
- Assert final queue contains exactly {CAPACITY_K} rows with zero null scores.
- Assert model Precision@{CAPACITY_K} achieves at least a 2.0x lift over baseline.

[OUTPUT FORMAT]
1. Complete, executable Python script with imports and error handling.
2. Markdown summary table comparing: Metric, Baseline Heuristic, Model, and Lift Ratio.
3. Markdown table of the top 10 queue candidates showing: `rank`, `content_id`, `priority_score`, and `primary_reason_code`.
```
