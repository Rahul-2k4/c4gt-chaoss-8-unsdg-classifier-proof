# CHAOSS #8 MiFi Demo Report

Status: private proof report for proposal drafting. Not public yet.

## Purpose

This report shows the evaluation path for the CHAOSS UN-SDG classifier MVP. It is not the finished 100-row DPG benchmark and it does not claim 85% accuracy. It shows that the proposal has a working proof direction: benchmark schema, metric logic, a completed real Aurora 10-row run, and one concrete payload-normalization issue.

## Demo Inputs

Current proof inputs:

- 10-row DPG Registry pilot CSV: `../DPG_10_ROW_PILOT.csv`
- metric notebook: `dpg-metric-demo.ipynb`
- metric summary: `dpg-metric-demo.md`
- real Aurora output anchor: `aurora-single-row-result.json`
- real Aurora pilot run outputs: `chaoss-dpg-10-row-aurora-real.csv`, `aurora-10row-raw-results.json`
- runtime/blocker log: `classifier-run-status.md`

## What Is Real vs Placeholder

Real:

- one direct Aurora provider call;
- one complete 10-row Aurora run at ~1 request/second;
- Aurora payload shape `{"text":"..."}`;
- top returned result for the African Storybook-style sample;
- bad metadata-shaped payload returned HTTP 500;
- backend dependency blocker is known.

Placeholder:

- aggregate accuracy numbers for the required 100-row benchmark;
- full 100-row DPG benchmark;
- final 85% accuracy result.

## Real Aurora Sample Set (10 Rows)

Provider: Aurora SDG classifier.

Working payload shape:

```json
{"text":"Provides open access to picture storybooks in the languages of Africa for children literacy, enjoyment and imagination."}
```

Observed top predictions from full 10-row run are stored in:

- `chaoss-dpg-10-row-aurora-real.csv` (normalized benchmark view)
- `aurora-10row-raw-results.json` (raw provider responses)

Quick summary from the run:

| Metric | Value |
| --- | --- |
| Rows | 10 |
| Exact match rate | `0.0` |
| Mean overlap score | `0.2456` |
| Top-k hit rate | `0.9` |

Representative top predictions:

| Sample | Top SDG | Score |
| --- | --- | --- |
| Aam Digital-style description | SDG 16 / Peace, Justice and strong institutions | `0.410952598` |
| Apache Fineract-style description | SDG 10 / Reduced inequalities | `0.511745512` |
| Bahmni-style description | SDG 9 / Industry, innovation and infrastructure | `0.262971163` |
| CKAN-style description | SDG 9 / Industry, innovation and infrastructure | `0.568605185` |
| ClimWeb-style description | SDG 13 / Climate action | `0.633245528` |
| African Storybook-style DPG description | SDG 4 / Quality Education | `0.951754928` |

Proposal implication: provider calls should receive one canonical text field, not raw project metadata fields.

## Metric Demo

The metric notebook currently demonstrates scoring logic with placeholder predictions. It computes:

- exact set match;
- overlap / Jaccard score;
- top-k hit;
- false positives;
- false negatives.

This matters because DPG Registry labels are multi-label. A project can map to several SDGs, so exact-set accuracy may be too strict. The proposal now uses a default target: at least 85% of rows should reach Jaccard overlap `>= 0.5` between predicted SDGs and DPG Registry SDGs. Exact-set match and top-k hit rate remain secondary metrics. Mentors can tighten the metric, but the proposal no longer waits for that answer before defining a plan.

## Payload Normalization Finding

Direct Aurora probe result:

- `{"text":"..."}` worked.
- `{"title":"...","abstract":"...","url":"..."}` returned HTTP 500.

Engineering implication:

```text
DPG row
  -> choose README/About/project text
  -> build canonical text payload
  -> call provider behind throttle/retry wrapper
  -> normalize labels/scores/errors
  -> write benchmark row
```

This should be implemented as a small backend service boundary: request normalizer, provider client, throttle/cache wrapper, and structured result envelope.

## Final DPG Benchmark Flow

```text
DPG Registry project
  -> README or website About source text
  -> canonical text payload
  -> Aurora / OSDG / sentence-transformer route
  -> normalized result envelope
  -> benchmark CSV
  -> Jupyter metrics
  -> mismatch report
  -> targeted accuracy improvements
  -> rerun benchmark
```

## Reviewer Takeaway

This proof does not complete the project. It shows the project can be made measurable. The strongest next proof is a small public benchmark/evaluation package after Rahul approves publication: schema, metric notebook, one real provider result, and payload-normalization note.
