# Classifier Run Status

Status: internal blocker/run log. Do not publish directly.

## Goal

Maintain accurate run log for real-vs-planned benchmark evidence.

## Current State

Available:

- repo cloned at `/Users/rahul/Desktop/CFGT/.tmp/UNSDG-classifier-tool`;
- frontend build passed in prior proof;
- backend source syntax passed in prior proof;
- 10-row benchmark CSV exists;
- metric notebook exists.
- real Aurora 10-row pilot run completed and stored in `evidence-pack/02-aurora-10row.csv` with raw responses in `evidence-pack/03-aurora-10row-raw.json`.

## 2026-05-03 Runtime Probe Evidence

Backend import checks from `/Users/rahul/Desktop/CFGT/.tmp/UNSDG-classifier-tool/backend`:

- `python3 -c "import flask, flask_cors, requests; print('core imports ok')"` failed with `ModuleNotFoundError: No module named 'flask'`.
- `python3 -c "import aurora_api; print('aurora_api import ok')"` failed with `ModuleNotFoundError: No module named 'requests'`.
- `python3 -c "import app; print('app import ok')"` failed with `ModuleNotFoundError: No module named 'requests'`.

Direct Aurora probe:

- Endpoint: `https://aurora-sdg.labs.vu.nl/classifier/classify/elsevier-sdg-multi`
- Working payload shape: `{"text":"..."}`
- Sample: African Storybook DPG-style description.
- Top returned prediction: SDG 4 / Quality Education, score `0.951754928`.
- Bad payload shape with `title`, `abstract`, and `url` returned HTTP 500. This supports a concrete proposal task: normalize classifier request payloads and add resilient error handling before batch runs.

Not yet complete:

- 100-row benchmark has not been executed yet;
- OSDG route needs `OSDG_TOKEN`;
- Aurora/OSDG rate limits still need wrapper implementation for safe large-batch execution.

## Safe Next Commands

Frontend:

```bash
cd /Users/rahul/Desktop/CFGT/.tmp/UNSDG-classifier-tool/frontend
npm run lint
npm run build
```

Backend setup:

```bash
cd /Users/rahul/Desktop/CFGT/.tmp/UNSDG-classifier-tool/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python3 app.py
```

Manual sample call:

```bash
curl -sS http://127.0.0.1:5000/api/classify_aurora \
  -H 'Content-Type: application/json' \
  -d '{"projectName":"African Storybook","projectUrl":"https://www.digitalpublicgoods.net/registry","projectDescription":"Provides open access to picture storybooks in the languages of Africa for children literacy, enjoyment and imagination."}'
```

## Proposal Wording

Use:

> I have completed a real Aurora 10-row pilot benchmark with raw response capture and metric summary. The next proof step is a full 100-row run and OSDG/local-model comparison after rate-limit wrapper hardening.

Avoid:

> I have benchmarked classifier accuracy on 10 DPG projects.

That would overclaim.
