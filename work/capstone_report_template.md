# Capstone Report — <your lane>

- **Author:** MUHAMMAD HASSAN RAZA QURESHI
- **Lane:**   MACHINE LEARNING
- **Repo:**   https://github.com/hassanqureshi46278-art/flyrank-Internship-ML
- **Date:**   july 2026

> Copy this file to `work/capstone_report.md` and fill it in as you build. Sections 1–8
> mirror the Pass / Needs-Work rubric axes, so nothing here is optional. Sections 0 and 9
> are **paper sections**: your deployed research paper must carry both, and they're here so
> you never rebuild them from memory at ship time.

## 0. Abstract

We asked which pages in a content portfolio should reach a human reviewer first, out of far more candidates than any team can check by hand. Using a [[N ROWS]]-page anonymized slice of the FlyRank ML Internship warehouse (fact_content_daily_performance, month=2026-03), we engineered staleness, demand, and position features and trained a Random Forest ranking model under a client-grouped validation split. Compared against a Week-4 hand-written baseline rule on the same split and metric, the model showed [[YOUR RESULT, e.g. "an R² of 0.XX vs the baseline's 0.XX (n=XXX test pages)"]]. The top-ranked predictor by permutation importance was [[TOP FEATURE]]. The output is offered as decision-support for a human content reviewer, not a causal claim about what refreshing any individual page will do.

## 1. Problem framing

**Unit of analysis:** one page (content_key), aggregated over one mid-panel month.

**Output:** a continuous priority score per page, ranked into a queue — not a binary label, since the real decision is who goes to the top of a capacity-limited list, not whether any single page is "good" or "bad."

**Decision this supports:** which existing pages get reworked (content update, refreshed metadata, added sections) in the next content sprint.

**Who acts on it:** a content/SEO strategist with limited hours each week, who needs a short, defensible shortlist rather than a spreadsheet of every page on the site.

**Cost of a wrong call:** a false positive (flagging a fine page) wastes writer hours and delays pages that genuinely needed the work; a false negative (missing a quietly decaying page) costs the client another cycle of lost visibility before anyone notices. Because writer time is the scarce resource, precision at the top of the queue matters more than accuracy across every page.

**Why ML helps:** a fixed if-statement rule treats staleness, demand, and position as independent and equally weighted; in practice they interact — a slightly-stale page with high demand may outrank either signal alone. A learned model can pick up that interaction instead of relying on a few hand-tuned thresholds.

## 2. Data safety

**Data used:** fact_content_daily_performance from FlyRank/internship-warehouse (Hugging Face), filtered to month=2026-03 — a mid-panel month, not the sealed final month (_sample, June 2026).

**Deliberately excluded, and why:**
- fact_content_query_90d (query-level table) — adds a join over salted, rare-tail-aggregated hashes with no benefit to a page-level score, and widens the leakage surface.
- dim_clients fields beyond gsc_data_start/ga4_data_start — anonymization terms restrict use of other client dimension fields.
- _sample table (June 2026) — sealed as held-out; never used to build features or tune the model, since it's the natural outcome window for any past-to-future label.
- ctr_28d (engineered, then dropped) — derived directly from the target column (avg_clicks_28d ÷ avg_impressions_28d); confirmed leaky in Section 4.
- trend_direction, trend_pct (if present in the warehouse) — label-derived, describing the very outcome the model is trying to anticipate. Confirmed absent from the final feature set. [[VERIFY THESE COLUMN NAMES AGAINST YOUR ACTUAL DESCRIBE OUTPUT]]

**Pseudonymous IDs:** content_key and client_key are used only as grouping/join keys — for aggregation and for the client-grouped split — never fed to any model as a feature.

**Confirmation:** no client names, page URLs, or raw query text appear anywhere in work/ — every identifier used is the dataset's own salted hash key. [[CONFIRM THIS AGAINST YOUR ACTUAL COMMITTED FILES]]
## 3. Baseline

**The rule:** a single weighted score — 0.6 × staleness_norm + 0.4 × volume_norm — combining two signals checked against real bucket tables first (staleness behind FlyRank's refresh flags, volume behind the quick-win logic). Outputs one score, one reason code (STALE_HIGH_DEMAND), and a binary action label.

**Why it's a fair comparison:** evaluated on the exact same client-grouped test split and the exact same metric (R², MAE against avg_clicks_28d) as the model in Section 4 — not a friendlier slice or metric.

**Baseline numbers:**
| Metric | Value |
|---|---|
| R² | [[BASELINE R²]] |
| MAE | [[BASELINE MAE]] |

Signal verdicts: staleness → [[CONFIRMED/OPPOSITE/MIXED/FALSE]]; volume → [[CONFIRMED/OPPOSITE/MIXED/FALSE]].

## 4. Model / analysis

**Method:** Random Forest Regressor. Chosen over a linear baseline because the signals plausibly interact, and because it provides permutation importance for interpretation without heavy tuning — unlike gradient boosting, which tends to overfit a single-month sample this size.

**Final feature list:** avg_impressions_28d, avg_position_28d, days_since_update, days_active_in_month.

**Left out on purpose:** ctr_28d (leaky — derived from the target); query-level features (unnecessary join, wider leakage surface); client_key/content_key (identifiers, used only for grouping).

**Target/proxy, one sentence:** avg_clicks_28d (mean daily clicks over the month) is used as a proxy for ongoing page value, since no observed "refresh worked" outcome exists in this snapshot.
## 5. Evaluation

**Split:** client-grouped (GroupShuffleSplit on client_key, 80/20, random_state=42) — not a naive random split, not yet time-aware. Grouping by client prevents one client's correlated pages from appearing on both sides of train/test, which would otherwise overstate performance. A naive random split was also run for comparison (see table) to make that overstatement visible rather than hidden.

**Metrics vs. base rate:** this is a regression task (predicting avg_clicks_28d), so there's no majority-class base rate in the classification sense — the honest analog is comparing model R² against a "predict the mean" baseline (R²=0) and against the rule-based baseline below, rather than reporting R² in isolation.

| Model | Split | n_test | R² | MAE |
|---|---|---|---|---|
| Baseline (rule score) | Client-grouped | [[n]] | [[R²]] | [[MAE]] |
| Random Forest | Naive random | [[n]] | [[R²]] | [[MAE]] |
| Random Forest | Client-grouped | [[n]] | [[R²]] | [[MAE]] |

**Error analysis:** the worst [[N]] predictions clustered around [[YOUR REAL PATTERN — e.g. "very low avg_impressions_28d, where the model has little signal to work with"]].

## 6. Interpretation

**Permutation importance (plain words):**
| Feature | Importance | What this means |
|---|---|---|
| [[FEATURE 1]] | [[VALUE]] | [[one-sentence plain-word interpretation]] |
| [[FEATURE 2]] | [[VALUE]] | [[...]] |
| [[FEATURE 3]] | [[VALUE]] | [[...]] |

**Surprises / negative results:** [[e.g., "the volume signal came back MIXED, not CONFIRMED, in Section 3's bucket check — reported here rather than dropped silently, since a well-understood 'no effect' is a valid result."]]

## 7. Recommendation

**Ranked actions:**
1. Refresh stale, high-demand pages first (STALE_HIGH_DEMAND archetype) — matches the clearest measured lever in FlyRank's own portfolio research.
2. Improve snippets on pages already ranking page-one (PAGE_ONE_HOLDER) — click-capture gains compound faster here than building new pages.
3. Monitor, don't touch, young/fresh pages (FRESH_YOUNG) — still in the natural growth window.
4. Re-audit signals before applying this queue to a new client vertical or month — verdicts in Section 3 were confirmed on one slice only.

**How a FlyRank editor uses this tomorrow:** pull the top 20–50 rows of the ranked queue, cross-check each against the human-review checklist (is it seasonal? under legal hold? does the reason code match the page?), and only then schedule the approved subset into the sprint. Nothing here auto-publishes.

**Confidence and limits, stated explicitly:** this ranking is directional, decision-support, and validated on one mid-panel month for the clients in this sample — not validated across time, across a different client vertical, or as a causal claim about what refreshing any specific page will do.

## 8. Reproducibility

**Fresh-clone commands:**
```bash
git clone https://github.com/hassanqureshi46278-art/flyrank-Internship-ML
cd flyrank-Internship-ML
pip install -r requirements.txt
```
Then open work/notebooks/capstone.ipynb in Colab, set the HF_TOKEN secret, and Run All.

**Random seeds:** random_state=42 used consistently across train_test_split, GroupShuffleSplit, RandomForestRegressor, and permutation_importance.

**Environment:** [[PASTE pip freeze HIGHLIGHTS OR requirements.txt — at minimum: pandas, duckdb, scikit-learn, matplotlib versions]]

**Sealed/holdout evaluation:** the final month (_sample, June 2026) was not used to build features, tune the rule, or train the model. [[IF YOU HAVE ACTUALLY RUN A HOLDOUT CHECK AGAINST IT: link the specific cell/script and the committed metrics file here. IF YOU HAVE NOT: say so plainly — an unclaimed sealed test is honest; a claimed one that isn't checkable in the repo is not.]]

## 9. Acknowledgments & data credit

Built on the **FlyRank ML Internship dataset**. Data source and program credit: [flyrank.ai](https://flyrank.ai).

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
