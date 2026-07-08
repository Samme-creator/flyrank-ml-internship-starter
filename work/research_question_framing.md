# Research Question Framing

## Lane
Lane 2: Refresh / Content Opportunity Scoring (Core lane)

**Why I chose this lane:** I picked this lane because I already worked 
with this exact type of problem in notebooks 01 and 02 — building a 
hand-written rule to flag pages for review, then comparing it against a 
decision tree and random forest model using Precision@K. Continuing in 
this lane lets me build directly on what I already understand, instead 
of starting from a completely new problem.

## Background
FlyRank's content review process needs a way to prioritize which pages 
get reviewed first, since a reviewer cannot manually check every page 
in a client's site. This project builds a ranked "review queue" that 
scores pages by how urgently they need attention, based on observable 
search and engagement signals.

## The Question
Which pages should be reviewed first for a content refresh — update, 
expand, merge, prune, or monitor — based on evidence that they are 
declining, stale, or under-performing relative to their visibility?

## Unit of Analysis
One row = one content page, identified by `content_hash_id` (warehouse 
release) or `content_id` (starter dataset). Each page is evaluated on 
its own, not aggregated at the client or site level.

## Data Sources
- Starter phase: `data/raw/content_refresh_anonymized.csv` (~30,000 
  pages) to build and test the pipeline end to end.
- Full capstone phase (pending mentor access): the warehouse release 
  `dim_content` joined with `fact_content_daily_performance`, to build 
  a stronger, future-looking label instead of the current-window proxy 
  used in the starter notebooks.

## Candidate Features (observable, pre-decision signals only)
- Visibility: `impressions_90d`, `sessions_90d`, average position, 
  position tier
- Freshness: `content_age_days`, `days_since_last_update`, freshness 
  tier
- Engagement: `ctr`, `engagement_rate`, `scroll_rate`
- Content shape: `word_count`, word-count tier, content type, main 
  intent
- Demand context: search volume, competition level, CPC

Excluded on purpose: any FlyRank product decision output (`health_score`, 
`priority_score`, `action_type`, refresh flags) — these are not shipped 
in the internship data, and even if reconstructed, they must never be 
used as a model feature or label, since that would just teach the 
model to copy an existing rule instead of finding real signal.

## The Output
A ranked list (a "review queue") of pages, sorted by a refresh-priority 
score (0–100), with:
- a predicted probability of decline (from the model)
- a reason code per page (for example: `stale_visible_page`, 
  `declining_with_demand`, `low_ctr_visible_page`)
- a confidence label (high-confidence flags require enough evidence — 
  minimum impressions and sessions — not just a high score alone)

## The Action Someone Could Take
A content editor or SEO reviewer works through the ranked list, 
starting from the top, and for each flagged page decides: update the 
content, expand it, merge it with a related page, prune/remove it, or 
leave it under monitoring. The reason code helps them understand why a 
page was flagged, so they're not just trusting a black-box number.

## Cost of a Wrong Recommendation
- **False positive** (flagging a healthy page): wastes reviewer time on 
  a page that didn't need attention. Low cost per instance, but adds 
  up if the queue is noisy.
- **False negative** (missing a declining page): the page keeps losing 
  traffic unnoticed until someone catches it another way — potentially 
  a bigger, delayed cost.
- Because review capacity is limited (a team might only review the top 
  20–50 pages), precision at the top of the ranking matters more than 
  overall accuracy. This is why the project will report Precision@20 
  and Precision@50 rather than a single accuracy number.

## Why Data or ML Helps (Not Just a Person Guessing)
With thousands of pages, no reviewer can check every page by hand on a 
regular basis. A simple hand-written rule (e.g. "flag pages that are 
old and still getting traffic") catches some obvious cases but misses 
subtler patterns. In the Week 1/2 notebooks, a random forest model 
reached Precision@50 of 0.74 versus 0.24 for the hand-written rule — 
roughly 3x more of the top 50 flagged pages were actually declining. 
That gap is the evidence that a learned model adds real value here, 
beyond intuition alone.

## Validation Plan
- Client-holdout split: pages from the same client are never split 
  across train and test, so the model isn't just memorizing a client's 
  quirks.
- Baseline first: compare every model against the transparent hand-rule 
  baseline, not just against a blank slate.
- Leakage audit: confirm no feature is calculated after the decision 
  point, and that the label isn't reconstructed from a product flag or 
  a future-window metric that overlaps the feature window.
- If time allows, move from the starter's current-window label 
  (`trend_direction == "down"`) to a future-looking label using the 
  warehouse's daily fact table, e.g.: features from the prior 90 days 
  → decline over the next 30 days.

## Caveats
- This is a decision-support tool, not a guarantee — a high score means 
  "worth reviewing first," not "confirmed problem."
- The starter label is a simple current-window proxy, not a true future 
  outcome; results from it should be treated as a beginner baseline, 
  not the final capstone claim.
- All claims will be framed as observed/directional, based on the data 
  available — not as proof of how Google's search algorithm works, and 
  not as proof that a refresh caused any recovery (that would require 
  a controlled experiment).
- All outputs will be public-safe: pseudonymized IDs and aggregated 
  metrics only, no raw URLs, client names, or query text.
