# Capstone Report

- **Author:** Syed Samiullah
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/Samme-creator/flyrank-ml-internship-starter
- **Date:** 2026-08-14

## 1. Problem Framing

**Decision supported:** which pages a content editor spends limited 
review time on first.

**Unit of analysis:** one content page, within one client, on one 
calendar day.

**Output:** a ranked score plus a reason code and suggested action per 
page (prioritize for refresh, monitor, targeted improvement, or no 
action).

**Action a human takes:** an editor reviews the top of the queue and 
decides whether to update, expand, merge, prune, or monitor each 
flagged page.

**Cost of a wrong call:** a false positive wastes a reviewer's limited 
time on a healthy page. A false negative lets a real decline go 
unnoticed until it's worse.

**Why data/ML helps:** no editor can manually check thousands of pages 
regularly. A model can combine several weak signals (position, 
volume, clicks) in ways a simple fixed rule cannot.

## 2. Data Safety

**Data used:** `fact_content_daily_performance`, slice `month=2026-03`, 
from the FlyRank internship warehouse (Hugging Face, gated release).

**Columns deliberately excluded, and why:**
- Any FlyRank product decision flag (`health_score`, `priority_score`, 
  `action_type`) - not present in this table by design, not 
  reconstructed.
- AI-referral columns - too sparse per the lane guide.
- `imp_total` and `imp_second_half` - removed after a leakage audit 
  found they could reconstruct the label (see Section 5).

**Leakage risks considered:** label-derived fields like `trend_pct` 
were never used as features. `client_hash_id` and `content_hash_id` 
are used only for grouping/joins, never as model features.

**Confirmation:** No client names, domains, URLs, or raw queries 
appear anywhere in `work/` - only pseudonymized hash IDs.

## 3. Baseline

A transparent rule: pages with at least 20 first-half impressions and 
strong average position, scored by first-half volume. Fair comparison 
because it uses the same underlying signals as the model and never 
references the label.

**Baseline results (client-holdout test set, base rate of declining 
pages ≈ 9-10% of test set):**

| k | Precision | Base rate |
|---|---|---|
| 20 | 0.00 | ~0.09 |
| 50 | 0.06 | ~0.09 |

## 4. Model / Analysis

**Method:** Random forest classifier (200 trees) - captures non-linear 
interactions between signals while staying interpretable.

**Feature list:** `imp_first_half`, `avg_position`, `avg_clicks`.

**Left out on purpose:** `imp_total`, `imp_second_half` (leakage), 
FlyRank product flags, AI-referral columns.

**Label, in one sentence:** a page is marked declining if its 
second-half-March impressions fell below 80% of its first-half rate.

## 5. Evaluation

**Split:** grouped by `client_hash_id` - pages from one client never 
appear in both train and test. Verified zero client overlap.

**Model vs. baseline, same split, same metric:**

| k | Baseline | Model | Lift over baseline |
|---|---|---|---|
| 20 | 0.00 | 0.35 | well above ~0.09 base rate |
| 50 | 0.06 | 0.42 | ~4.7x the ~0.09 base rate |

**Error analysis:** 2,113 of 16,193 test rows (~13%) are false 
positives. Median false-positive page sits near position 12.5 with 
modest first-half impressions (median 154).

**Leakage caught:** an early version scored a suspicious 1.0 precision 
at both k=20 and k=50 - traced to the baseline formula referencing the 
label directly, and `imp_total` being reconstructable into the label's 
source column. Both removed; result above is the honest number.

## 6. Interpretation

**Feature importances:** avg_position (58.4%), imp_first_half (34.7%), 
avg_clicks (6.8%). The model relies mostly on ranking position, using 
volume as a secondary check, largely ignoring click behavior.

**Negative/surprising result:** re-running under a naive random split 
gave nearly identical Precision@20 (0.35) and a *lower* Precision@50 
(0.32 vs. the grouped split's 0.40) - opposite of the usual assumption 
that random splits inflate scores. Reported honestly as a "no clear 
effect" finding rather than forced to match the hypothesis.

## 7. Recommendation

**Ranked action tiers:** high_risk_declining (9.1%, prioritize for 
refresh), moderate_risk_visible (5.8%, monitor closely), 
low_position_opportunity (18.2%, targeted relevance improvement), 
low_risk (66.9%, no action).

**How an editor uses this tomorrow:** open the ranked queue, start 
from the top of `high_risk_declining`, manually review each page 
before taking any action.

**Confidence and limits:** Precision@50 of 0.42 means roughly 6 in 10 
flagged pages in the top 50 will NOT actually be declining - this is 
decision-support, not certainty. Validated on one month of one client 
slice only.

## 8. Reproducibility

**Environment:** Google Colab, Python 3, key packages: `duckdb`, 
`huggingface_hub`, `scikit-learn`, `pandas`.

**Random seed:** 42, fixed throughout.

**To re-run from a fresh clone:**
1. Clone the repo
2. Create a Hugging Face read token, request access to 
   `FlyRank/internship-warehouse`
3. Open `work/notebooks/w03_data_contract.ipynb` through 
   `w07_action_playbook.ipynb` in order, run top to bottom
4. Metrics JSONs at `work/outputs/*.json` are the receipts these 
   numbers trace back to

---

**Claims checklist confirmed:**
- Language throughout: observed / measured / directional / 
  decision-support - confirmed
- Base rate reported alongside precision@K (Section 3, ~9%) - confirmed
- No causal claims without an experiment - confirmed (Section 7 states 
  decision-support, not certainty)
- No claim about Google's algorithm - confirmed
- No client-identifying details anywhere - confirmed
- Numbers match a fresh re-run - confirmed via committed metrics JSON
