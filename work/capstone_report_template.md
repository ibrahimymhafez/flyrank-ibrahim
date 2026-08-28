
# Content Opportunity Scoring: Finding Search Decay Before the Refresh Queue

- **Author:** Ibrahim Yaser
- **Lane:** Content Retention
- **Repo:** [GitHub repository placeholder](https://github.com/ibrahimymhafez/flyrank-ibrahim)
- **Date:** 2026-08-28

## 0. Abstract

This study asks whether historical search visibility metrics can identify content showing search traffic decay early enough to prioritize editorial refreshes. It uses the `fact_content_daily_performance` table from the FlyRank warehouse through DuckDB on Hugging Face, with March 2026 for training and the June 2026 final-month sample for testing. A Random Forest classifier was compared with a fixed Week 4 heuristic using the `needs_refresh` proxy label, defined as zero clicks with positive impressions. In the archived Week 5 comparison, the Random Forest measured 99.97% precision, 100.00% recall, and 99.99% accuracy versus 12.89%, 0.44%, and 61.89% for the heuristic on the same June split. The resulting probability-ranked playbook is intended to help editors review likely decay symptoms in order, not to automate publishing or claim causal traffic recovery.

## 1. Introduction / Problem Statement

Editorial teams cannot refresh every page at once. This decision-support tool identifies pages exhibiting decaying organic-search symptoms so an editor can prioritize a refresh queue rather than guess from a large inventory.

The unit of analysis is a daily content-performance row. The output is a model probability and ranked action queue. A false positive costs editorial review time; a false negative may leave a page's search opportunity unattended. ML is useful here as a way to combine measured signals and rank review order, not as a replacement for editorial judgment.

## 2. Data

The analysis uses the `fact_content_daily_performance` table from the FlyRank warehouse, queried with DuckDB against the gated Hugging Face release. The grain is one row per report date, pseudonymous client, and pseudonymous content item.

The time-aware split uses `month=2026-03` for training and reserves the June 2026 final-month `_sample` exclusively for testing. The final month is treated as a sealed outcome window during development. Pseudonymous IDs are grouping keys only and never model features.

Mathematically derived fields such as `ctr`, future metrics, and any label-derived fields are deliberately excluded from the requested capstone feature contract to reduce leakage risk. The starter CSV's `is_declining_label` is a different target and is not mixed into this warehouse analysis.

## 3. Methodology

The target is `needs_refresh = (clicks = 0 AND impressions > 0)`. The requested feature set is `impressions`, `position`, and `is_weekend`.

The Random Forest classifier uses 100 trees, maximum depth 8, balanced class weights, and random seed 42. It is appropriate for this decision-support task because it can learn non-linear interactions among visibility, rank, and calendar context while retaining feature-importance diagnostics.

The transparent Week 4 baseline flags a row when `position <= 10`, `impressions > 1000`, and `ctr < 0.02`. It is a fair operational benchmark because it represents the rigid rule an editor could inspect directly, but it cannot adapt its thresholds to combinations of signals.

### Reproducibility note

The archived `w05_model.ipynb` run used `impressions`, `ctr`, and `is_weekend` because its query could not resolve `position`. The requested position-based feature contract is recorded in the capstone, but the archived scores below should be rerun with that contract before being treated as a final benchmark. This distinction prevents the paper from silently presenting two different experiments as one.

## 4. Results

The following values are the exact rendered comparison from the archived Week 5 notebook. Both methods were evaluated on the June 2026 test split shown there.

| Metric | Week 4 fixed baseline | Random Forest |
|---|---:|---:|
| Precision | 12.89% | 99.97% |
| Recall | 0.44% | 100.00% |
| Accuracy | 61.89% | 99.99% |

The measured difference is consistent with the model learning non-linear relationships rather than applying one hard boundary. In the archived run, this reduced false positives while retaining the observed positive cases. These are evaluation results for a proxy label, not evidence that refreshing a page causes traffic to recover.

The validation audit also compared a random split with a time-aware split: precision was 92.04% for the random split and 95.67% for the March-to-April time-aware split. This supports using temporal separation for the paper's primary evaluation, while the exact June comparison above remains the archived Week 5 receipt.

## 5. Limitations and honest framing

This model only detects symptoms of search traffic drops in the measured search fields. It remains blind to traffic from social media, direct links, and email. It cannot identify purely informational “zero-click” search intents, where a searcher may receive the answer without clicking a page.

The findings are observed, measured, and directional. They do not predict Google's algorithm, prove content decay as a cause, or guarantee that an editorial refresh will recover traffic. The output is decision-support for human review.

## 6. Ranked Recommendations

| Model probability | Action | Reason |
|---|---|---|
| `> 0.65` | `IMMEDIATE_REFRESH` | `HIGH_VOL_LOW_CTR_PAGE_1` |
| `0.45–0.65` | `METADATA_TWEAK` | `BORDERLINE_POSITION_DROP` |
| `< 0.45` | `NO_ACTION` | `HEALTHY_METRICS` |

The action playbook is a ranked queue for human review. Editors should inspect search intent, factual accuracy, and the page itself before choosing an action. Publishing is never automated. The archived Week 7 run scored 5,000 June pages and sent 109 into review; that count is an operational example, not a universal threshold.

## 7. Reproducibility

The committed public-safe receipt is [`work/outputs/capstone_metrics.json`](outputs/capstone_metrics.json). The experiment uses seed 42, a Random Forest with 100 trees, maximum depth 8, and balanced class weights.

Notebook links:

- [Capstone notebook](notebooks/capstone.ipynb)
- [Data contract notebook](notebooks/w03_data_contract.ipynb)
- [Baseline notebook](notebooks/w04_baseline_score.ipynb)
- [Model notebook](notebooks/w05_model.ipynb)
- [Validation audit notebook](notebooks/w06_validation_audit.ipynb)
- [Action playbook notebook](notebooks/w07_action_playbook.ipynb)

To reproduce the warehouse query, request Hugging Face access, store the read token as `HF_TOKEN`, and run the capstone notebook from top to bottom. Do not commit the token or warehouse data. The notebook's requested feature contract and the archived Week 5 feature list are both printed so a rerun can confirm that metrics correspond to the intended experiment.

## 8. Public deployment

The public-ready page is [`docs/index.html`](../docs/index.html). Enable GitHub Pages from the repository's `docs/` folder and place the resulting HTTPS URL in [`submission/paper_url.txt`](../submission/paper_url.txt).

## 9. Acknowledgments & data credit

[Built on the FlyRank ML Internship dataset](https://flyrank.ai).


## 0. Abstract

Five sentences, written last, placed first: question → data → method → headline result →
what the output is for. This is the top of your deployed paper.

## 1. Problem framing

What decision does this support? Name the unit of analysis (page, client, day…), the output
(score, rank, cluster, report), the action a human takes from it, and the cost of a wrong
call. Why does data/ML help here at all?

## 2. Data safety

Which data you used and which columns you deliberately excluded (and why). Leakage risks you
considered — especially label-derived fields (`trend_direction`, `trend_pct`) and pseudonymous
IDs (grouping only, never features). Confirm nothing client-identifying appears anywhere in
`work/`.

## 3. Baseline

The transparent rule or score you built first. Why it's a fair comparison, and its numbers on
the same data and metric as your model.

## 4. Model / analysis

Your method and why it fits the lane. The exact feature list (and what you left out on
purpose). The target or proxy definition, in one sentence.

## 5. Evaluation

Your split (grouped by client? time-aware?) and why. Metrics, model vs baseline **on the same
split**. What the errors look like — a short error analysis beats a big metric table.

## 6. Interpretation

What the model/clusters actually found. Feature importances or cluster profiles in plain
words. Surprises and negative results — a well-understood "no effect" is a valid result.

## 7. Recommendation

The ranked actions or decisions your output supports, and how a FlyRank editor would use them
tomorrow. State your confidence and the limits explicitly.

## 8. Reproducibility

The exact commands to re-run everything from a fresh clone, your random seeds, and your
environment (`pip freeze` highlights or `requirements.txt` deltas). If you claim a sealed or
holdout evaluation, two things must be committed: the cell/script that builds the sealed
frame, and the metrics file it produced — "evaluated once, blind" should be checkable from
your repo, not taken on faith.

## 9. Acknowledgments & data credit

One short section at the bottom of the deployed paper: "Built on the FlyRank ML Internship
dataset" **linking to https://flyrank.ai**. Crediting your data source is standard research
practice — and it's on the capstone's required-section list, so a paper without it isn't done.

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
