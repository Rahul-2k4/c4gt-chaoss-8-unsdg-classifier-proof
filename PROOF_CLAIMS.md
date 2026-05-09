# Proof Claims — CHAOSS #8 UN SDG Classifier

## What We Claim

- **Aurora 10-row pilot** with real classifier outputs on DPG Registry data — proving the classify + export pipeline works on real input
- **UI classify/export screenshots** showing the frontend classify flow, JSON export, and CSV export working as designed
- **Metrics summary** with accuracy notes and a clear achieved-vs-target boundary — separating what the benchmark proved from what the full 100-row run will validate
- **API base URL blocker note** documenting the localhost dependency and proposing configurable base URLs as the first deployment step
- **Benchmark plan** defining the 100-row evaluation scope, source provenance (DPG Registry), target threshold (85%), and what counts as evidence
- **Clear proof boundaries** separating achieved proof (10-row pilot, UI/export) from planned proof (100-row benchmark, public deployment)

## What We Do NOT Claim

- No upstream PRs merged
- No full 100-row benchmark run
- No public deployment confirmed
- No 85% accuracy claimed
- No mentor approval

## Evidence

- `MIFI_PROTOTYPE/index.html` — interactive prototype
- `screenshots/prototype.png` — visual snapshot
- `proof_packet/` — Aurora CSV, raw JSON outputs, metrics summary, UI screenshots, API base URL note, benchmark plan
- `proof_packet/screenshots/` — pipeline, classify, export screenshots

## Claim Boundary

Local UI/API proof and 10-row Aurora pilot exist. 100-row benchmark and 85% target are not yet achieved.
