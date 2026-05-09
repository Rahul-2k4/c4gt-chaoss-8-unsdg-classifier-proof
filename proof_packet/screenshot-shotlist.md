# CHAOSS #8 Screenshot Shotlist

Status: private visual-evidence checklist for proposal.

## Core Shots (Use These)

1. `01-proof-index.png`
   - Source: `proof_packet/README.md` evidence table.
   - Purpose: shows proof breadth and honesty (real vs pending).

2. `02-aurora-10row-csv.png`
   - Source: `proof_packet/chaoss-dpg-10-row-aurora-real.csv`.
   - Purpose: proves real pilot predictions exist for all 10 rows.

3. `03-aurora-raw-json.png`
   - Source: `proof_packet/aurora-10row-raw-results.json`.
   - Purpose: proves reproducibility and provider-level trace.

4. `04-metric-summary.png`
   - Source: `proof_packet/mifi-demo-report.md` run summary table.
   - Purpose: shows measured baseline (`exact`, `overlap`, `top-k`) without overclaim.

5. `05-architecture-flow.png`
   - Source: proposal current/target architecture blocks.
   - Purpose: shows system understanding, not only benchmark output.

## Optional Shots

1. `06-payload-normalization-finding.png`
   - Source: payload note in `mifi-demo-report.md`.
   - Purpose: shows concrete integration bug found (`text` payload vs metadata payload).

2. `07-rate-limit-design.png`
   - Source: `proof_packet/rate-limit-wrapper-prototype.md`.
   - Purpose: shows operational reliability planning for 1 req/sec constraint.

## Caption Rule

- Keep captions factual and short.
- Never claim 100-row or 85% achieved.
- Safe caption template:
  - `“Private proposal proof: 10-row Aurora pilot baseline (2026-05-04).”`
