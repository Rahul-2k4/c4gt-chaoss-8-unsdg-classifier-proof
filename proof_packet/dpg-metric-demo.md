# DPG Metric Demo

Status: metric prototype using the 10-row pilot CSV. Placeholder predictions are included only to demonstrate scoring. They should be replaced by real classifier outputs after the backend runtime is configured.

## Metrics

For each DPG row:

- `exact_match`: predicted SDG set equals registry SDG set.
- `overlap_score`: Jaccard overlap, computed as `|predicted ∩ registry| / |predicted ∪ registry|`.
- `top_k_hit`: at least one predicted SDG appears in the registry SDG set.
- `false_positives`: predicted SDGs not listed in the registry row.
- `false_negatives`: registry SDGs not predicted by the tool.

## Why This Matters

Issue #8 asks for 85% accuracy against DPG Registry SDG relevance. Exact set match may be too strict for multi-label SDG relevance because one project can map to many SDGs and classifier confidence may be useful even when the full label set is incomplete.

The proposal should ask mentors whether the target metric should be:

1. exact set match;
2. partial overlap threshold;
3. top-k relevance;
4. mentor-reviewed relevance score.

## Notebook Logic

```python
import pandas as pd

def parse_sdgs(value):
    if pd.isna(value) or not str(value).strip():
        return set()
    return {part.strip().upper() for part in str(value).split(";") if part.strip()}

df = pd.read_csv("../chaoss-dpg-10-row-pilot.csv")
df["registry_set"] = df["registry_sdgs"].apply(parse_sdgs)
df["predicted_set"] = df["predicted_sdgs"].apply(parse_sdgs)

def overlap(row):
    union = row.registry_set | row.predicted_set
    if not union:
        return 0.0
    return len(row.registry_set & row.predicted_set) / len(union)

df["computed_exact_match"] = df["registry_set"] == df["predicted_set"]
df["computed_overlap_score"] = df.apply(overlap, axis=1)
df["computed_top_k_hit"] = df.apply(
    lambda row: bool(row.registry_set & row.predicted_set),
    axis=1,
)
df["false_positives"] = df.apply(
    lambda row: "; ".join(sorted(row.predicted_set - row.registry_set)),
    axis=1,
)
df["false_negatives"] = df.apply(
    lambda row: "; ".join(sorted(row.registry_set - row.predicted_set)),
    axis=1,
)

summary = {
    "rows": len(df),
    "exact_match_rate": df["computed_exact_match"].mean(),
    "mean_overlap_score": df["computed_overlap_score"].mean(),
    "top_k_hit_rate": df["computed_top_k_hit"].mean(),
}
summary
```

## Proposal Implication

The benchmark is more than a spreadsheet deliverable. It becomes the control loop for the project:

1. run baseline predictions;
2. score exact/partial/top-k metrics;
3. group mismatches by cause;
4. implement targeted preprocessing/post-processing/API reliability improvements;
5. rerun the benchmark and report before/after behavior.
