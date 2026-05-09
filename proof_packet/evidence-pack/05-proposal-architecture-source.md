# Making the CHAOSS UN-SDG Classifier Public, Measurable, and Production-Ready

Status: final draft for review. Personal details, public proof links, and mentor answers should be confirmed before submission.

## Project Metadata

| Field | Value |
| --- | --- |
| Applicant | Rahul Tripathi |
| Program | Code for GovTech DMP 2026 |
| Organization | CHAOSS / CHAOSS UN SDG Working Group |
| Project | UN-SDG Classifier MVP |
| Issue | https://github.com/chaoss/UNSDG-classifier-tool/issues/8 |
| Repository | https://github.com/chaoss/UNSDG-classifier-tool |
| Mentor | David Lippert |
| Project Size | DMP coding period |

## Contact Information

| Field | Value |
| --- | --- |
| Name | Rahul Tripathi |
| Email | rahultripathi7009@gmail.com |
| GitHub | https://github.com/Rahul-2k4 |
| LinkedIn / Portfolio | https://linkedin.com/in/rahul-tripathi-ai / https://rahul2k4.vercel.app |
| Timezone | IST (UTC+5:30) |

## About Me

I work across TypeScript, Python, backend APIs, and evaluation-heavy AI systems. For this project, the useful overlap is broader than a Next.js frontend and Flask backend. I can turn an AI classification feature into something measurable: a benchmark, a scoring method, a deployment path, and a report that maintainers can inspect.

I have already inspected the UN-SDG classifier locally. The frontend production build passes, backend source files pass syntax parsing, and the current frontend API client has a concrete deployment blocker: it calls `http://127.0.0.1:5000/` directly. I also created a 10-row DPG Registry pilot benchmark and a small metric prototype so the 85% accuracy target can be discussed in precise terms instead of treated as a vague model-improvement goal.

## Executive Summary

The UN-SDG classifier already has a useful foundation: a Next.js frontend, a Flask backend, multiple classifier routes, and JSON result export. The missing part is MVP readiness.

My proposal is to make the tool public, measurable, and reliable for DPG Registry evaluation. I am treating the issue as one connected product problem, not as separate hosting, notebook, and model tasks. It addresses every requirement in the idea and sets defaults for the unresolved parts so the project does not wait on mentor replies.

I will focus on four linked outcomes:

1. a no-cost public deployment with CI/CD;
2. a repeatable 100-project DPG Registry benchmark;
3. an accuracy analysis notebook with clear metrics and mismatch categories;
4. API and output hardening so results can be generated without overloading Aurora or OSDG services.

The end result should not be a black-box classifier demo. It should be a public tool plus evidence: benchmark data, metric definitions, before/after results, and documentation that lets CHAOSS maintainers understand what improved and what still needs human review.

## Idea Requirements Covered

Issue #8 asks for a public MVP of the UN-SDG classifier with 100 DPG Registry test results, 85% accuracy, CI/CD, no paid hosting, JSON or YAML output, and rate-limit-safe use of Aurora and OSDG APIs. I read the issue as requiring all of the following, and I map each one to a concrete deliverable and default decision:

| Idea requirement | Proposal response | Default if mentor guidance is delayed |
| --- | --- | --- |
| Decide hosting platform with no budget | Week 1 deployment audit with GitHub Pages checked first and free backend options evaluated | Primary: GitHub Pages for static frontend + PythonAnywhere free Flask backend if dependency size fits; fallback: Vercel static/frontend + Vercel Flask function if bundle size fits |
| Keep GitHub Pages as first preference where possible | Static frontend path stays compatible with GitHub Pages if the backend URL is configurable. Flask/model serving is handled separately because GitHub Pages is static hosting | Use GitHub Pages for frontend unless Next.js export constraints block it |
| Test Digital Public Goods Registry projects | Build a 10-row pilot first, then a 100-row benchmark using DPG project text and official SDG relevance labels | Use the first 100 DPG registry projects with accessible README/About/project text; record exclusions |
| Compare predicted SDGs with DPG Registry SDG relevance | Spreadsheet fields include registry SDGs, predicted SDGs, confidence scores, overlap, false positives, false negatives, and mismatch notes | Use DPG Registry SDG Relevance as ground truth and report caveats row-by-row |
| Search README or About text for each project | Benchmark process records the exact source text used for each project so results can be reproduced and disputed | Prefer README; use website About text if README is missing or too sparse |
| Analyze the spreadsheet using Jupyter | Notebook computes exact-set match, overlap/Jaccard score, top-k hit rate, false positives, false negatives, and mismatch categories | Notebook is required output, not optional analysis |
| Improve tool accuracy | Improvements are driven by benchmark failures: source-text selection, thresholding, label normalization, model/API disagreement, and dataset/model survey | First accuracy lever: better source-text selection + label normalization before model changes |
| Search for dataset or relevant models | Dataset/model survey is a bounded required task tied to measured failure modes | Search only datasets/models that address observed DPG mismatch categories |
| JSON or YAML output | Stable JSON is the required output, with YAML as a small additional export if maintainers want it | Commit to stable JSON first; add YAML if it improves maintainer handoff |
| Aurora and OSDG one request per second | Shared provider wrapper handles throttling, retries, timeouts, and optional cache for benchmark runs | Enforce one request per provider per second; validate OSDG with token or mocked provider |
| CI/CD to production | GitHub Actions or equivalent checks cover frontend build/type checks, backend syntax/tests, and deployment docs | CI must run frontend build, backend import checks, provider-wrapper tests, and notebook smoke checks |

## Problem Statement

The main challenge is bigger than "improve the model." The current project needs the pieces around the model that make it usable:

- the frontend must call a deployable backend, not localhost;
- the backend must handle external classifier APIs safely;
- the 100-row benchmark must be repeatable;
- the 85% target must be tied to a metric that makes sense for multi-label SDG relevance;
- output export must be verified and documented;
- deployment must fit the no-paid-services constraint.

SDG classification is multi-label. A DPG Registry entry can have many SDGs, while a classifier may return fewer high-confidence labels. My default official metric will be the share of benchmark rows with Jaccard overlap of at least 0.5 between predicted SDGs and DPG Registry SDGs. I will also report exact-set match and top-k hit rate so maintainers can see whether the tool is getting the right SDG family even when the full label set is incomplete.

## Current Architecture

The current system is roughly:

```text
Next.js frontend
  -> Flask backend
    -> Aurora API classifier
    -> OSDG API classifier
    -> sentence-transformer description classifier
    -> sentence-transformer URL classifier
    -> JSON result download
```

Current codebase findings from local inspection:

| Area | Finding | Proposal Implication |
| --- | --- | --- |
| Frontend | Next.js frontend builds locally | UI can be hosted separately if API URL is configurable |
| API client | `frontend/services/api.ts` hardcodes `http://127.0.0.1:5000/` | Public deployment needs environment-based API configuration |
| Backend | Flask routes exist for Aurora, OSDG, and sentence-transformer classifiers | Work should harden and normalize existing routes, not rebuild from scratch |
| Output | JSON download already exists in `results.tsx` | JSON should be verified/hardened; YAML should be added or confirmed with mentors |
| External APIs | OSDG route needs `OSDG_TOKEN`; Aurora/OSDG have rate-limit constraints | Benchmark runner needs throttling, retry/backoff, and caching |
| API payloads | A direct Aurora probe worked with `{"text":"..."}`, while sending project metadata fields directly returned HTTP 500 | Backend and benchmark runner need payload normalization before provider calls |
| Dependencies | Frontend install reported vulnerabilities in local notes | Dependency/security audit should be part of MVP hardening |
| Benchmark | 10-row DPG pilot scaffold exists | It should become the first step toward the required 100-row benchmark |

## Current Issue Discussion And Differentiation

Several contributors have already commented on issue #8. My edge is measurable benchmark design plus provider-call evidence, not only deployment configuration.

## Pre-Submission Evidence And MiFi Demo

I have already done a small qualification pass so this proposal is based on the live project surface, not only the issue text.

Current proof is deliberately limited: a real Aurora 10-row pilot run is complete, OSDG rows remain 0 because token/runtime setup is pending, no full 100-row aggregate run yet, and no public artifact posted yet.

| Evidence | What it proves | Status |
| --- | --- | --- |
| Frontend production build note | The Next.js frontend is a realistic static-hosting candidate after API configuration is fixed | Verified in local proof notes |
| Backend syntax/import probe | The backend code path exists, but the current local Python environment is missing `flask` and `requests` | Blocked until backend dependencies are installed |
| 10-row DPG pilot CSV | The DPG benchmark can be represented as a repeatable table instead of a one-off manual check | Scaffold ready and real Aurora run completed in proof packet |
| Metric notebook and Markdown summary | The 85% target needs an explicit metric: exact set match, overlap/Jaccard, top-k hit, or mentor-reviewed relevance | Prototype ready |
| Direct Aurora API call | Aurora can classify a DPG-style project description when the payload is normalized to `{"text":"..."}` | One real sample verified |
| Bad Aurora payload test | Sending `title`, `abstract`, and `url` directly returned HTTP 500 | Shows the need for request normalization and error handling |
| Rate-limit wrapper note | Aurora/OSDG batch calls need throttling, retries, timeouts, and cache support | Design prototype ready |

The MiFi proof packet is intentionally small. It is not a fake finished product. It shows the evaluation path I will expand during DMP:

```text
DPG Registry row
  -> chosen README/About/project text
  -> canonical text payload
  -> provider call through Aurora/OSDG/sentence-transformer route
  -> normalized result envelope
  -> benchmark spreadsheet
  -> Jupyter metrics and mismatch report
```

The current Aurora sample set includes the full 10-row pilot, including an African Storybook row that returned SDG 4 / Quality Education with score `0.951754928`. The next proof step is extending this to the 100-row benchmark and adding OSDG/local-model comparisons after backend setup and payload normalization.

## What Works vs What Needs Engineering

| Surface | Current State | Evidence | DMP Work |
| --- | --- | --- | --- |
| Next.js frontend | Existing UI and result flow are already present | Frontend build proof and source inspection | Make API base URL environment-driven; verify hosted frontend can reach backend |
| Flask backend routes | Classifier routes exist for Aurora, OSDG, and sentence-transformer paths | Backend source inspection and import probe | Install/runtime verification, route tests, structured errors |
| JSON/YAML output | JSON download exists; YAML is not yet confirmed | `results.tsx` source finding | Verify JSON schema; add YAML only if mentors want both |
| Aurora provider | Direct text payload works | Real API result for one DPG-style sample | Normalize payloads, add provider client, errors, throttling |
| OSDG provider | Route exists but needs `OSDG_TOKEN` | Repo/runtime notes | Keep token-gated path explicit; test when token is available |
| DPG benchmark | 10-row schema exists and Aurora run is complete | Pilot CSV + Aurora raw JSON | Expand to 100 rows with source-text notes and official registry labels |
| Accuracy notebook | Metric prototype exists | Notebook and Markdown summary | Run baseline/final metrics, mismatch categories, before/after report |
| No-cost deployment | Frontend can be static; backend cannot run on GitHub Pages | Architecture/runtime finding | Decide split hosting path and document free-tier constraints |
| CI/CD | Not deploy-ready yet | Issue requirement and local proof notes | Add frontend/backend checks and deployment documentation |
| API rate limiting | Needed for Aurora/OSDG batch runs | Issue requirement and wrapper prototype | Provider service layer with one-request-per-second throttling, retries, cache, and result envelopes |

Backend implementation should stay simple: a request normalizer builds one canonical text field, provider clients call Aurora/OSDG behind a throttle, and routes return a structured result envelope with provider name, labels, scores, source text metadata, and error details when a provider fails. This keeps benchmark runs reproducible and prevents one API failure from corrupting the whole spreadsheet.

## Target Architecture

```text
Public frontend
  -> configurable API base URL
    -> no-cost Flask backend
      -> classifier router
        -> Aurora / OSDG / sentence-transformer models
      -> rate-limit queue and retry wrapper
      -> result normalizer
      -> benchmark runner and cache
      -> JSON/YAML export
      -> CI/CD and deployment docs
```

This keeps the project close to the existing stack while fixing the MVP gaps. I do not plan to introduce paid services or a large rewrite unless the deployment audit proves the current architecture cannot meet the issue constraints.

## Proposed Implementation Tracks

### Track 1: Deployment and CI/CD

I will start with a concrete no-cost deployment path. GitHub Pages is suitable for a static frontend, but it cannot run the Flask backend or model dependencies because it serves static files. My primary architecture is GitHub Pages for the frontend and PythonAnywhere free hosting for the Flask backend if the dependency set fits its free-account limits. My fallback is Vercel for the frontend and a Vercel Flask function only if the Python bundle stays within Vercel's function limits.

Early work:

- replace hardcoded frontend API URL with environment-based configuration;
- verify the backend dependency size against the chosen free host before committing to that host;
- document local and production API configuration;
- add CI checks for frontend build/lint and backend syntax/tests;
- document deployment steps and rollback notes.

### Track 2: DPG Registry Benchmark

I will build the benchmark in two stages:

1. 10-row pilot for mentor review of schema and metrics;
2. 100-row benchmark using the default sampling rule unless mentors request a different sample.

Benchmark fields:

- project name;
- DPG Registry URL;
- source text used for classification;
- registry SDGs;
- predicted SDGs;
- confidence scores;
- exact match;
- overlap score;
- top-k hit;
- mismatch note.

The benchmark should become a regression artifact, not a one-time spreadsheet.

Default sampling rule: use the first 100 DPG Registry projects with accessible README, website About, or registry description text; record exclusions and source-text choices explicitly. This matters because the issue allows README text or website About text. If the classifier gets weak input, the mismatch should not be counted as a model failure without saying so.

### Track 3: Accuracy Analysis

I will create a Jupyter notebook that computes:

- exact-set match;
- Jaccard/overlap score;
- top-k hit rate;
- false positives;
- false negatives;
- mismatch categories.

The notebook will group failures by cause: weak source text, missing README/about context, broad platform descriptions, threshold issues, API/model disagreement, or genuinely ambiguous SDG mapping.

Primary target: at least 85% of rows should reach Jaccard overlap `>= 0.5` between predicted SDGs and DPG Registry SDGs. Exact-set match and top-k hit rate will be reported as secondary metrics. If mentors choose a stricter exact-set target, I will keep the same notebook and report the gap honestly with mismatch categories.

### Track 4: API Reliability

Aurora and OSDG should not be called in a tight loop during benchmark runs. I will add a shared wrapper for external classifier calls:

- one request per provider per second;
- retry/backoff for transient failures;
- timeout with clear error messages;
- optional cache for benchmark runs;
- structured response envelope for frontend display and notebook analysis.

I will normalize provider payloads before calling external APIs. My current probe showed Aurora accepts a simple text payload, while project metadata-shaped payloads can fail. The benchmark runner should therefore create one canonical text field from README/About/project metadata before any provider call.

OSDG-specific validation depends on token access. Until then, the wrapper and throttle logic will be provider-agnostic, verified against Aurora and mocked OSDG responses, then rerun against OSDG when credentials are available.

### Track 5: Output and UX Hardening

The frontend already has JSON download behavior. I will verify and document the JSON structure as the required export. YAML will be a small follow-up if maintainers want a second format for handoff or automation. I will also improve user-facing result clarity where needed: confidence labels, low-confidence warnings, and source/model notes.

## Deliverables

### Required Deliverables

| Deliverable | Evidence |
| --- | --- |
| Public no-cost MVP deployment | GitHub Pages frontend plus PythonAnywhere Flask backend if feasible; documented fallback if dependency limits block it |
| CI/CD | GitHub Actions or equivalent workflow, build/test status |
| 100 DPG Registry benchmark | CSV/spreadsheet with inputs, registry labels, predictions, scores, notes |
| Accuracy analysis | Jupyter notebook and summary report |
| Accuracy improvement work | before/after metrics and mismatch categories |
| Dataset/model survey | Short report tied to measured failure modes, not an unrelated model hunt |
| JSON/YAML output | verified JSON export, with YAML added if maintainers want a second format |
| Rate-limit-safe API calls | wrapper implementation, test/proof, benchmark run notes |
| Documentation | setup, deployment, benchmark, and maintainer handoff docs |

### Optional / Stretch Deliverables

| Deliverable | Condition |
| --- | --- |
| Additional classifier dataset survey | If benchmark shows current model needs broader training/evaluation data |
| Small public demo video | If deployment is stable before final evaluation |
| Expanded benchmark beyond 100 rows | If 100-row benchmark is complete and mentors want broader coverage |

## Pre-Submission Proof

I have started a small proof bundle so the proposal is based on actual inspection:

- local repo inspection and build notes;
- 10-row DPG Registry pilot CSV;
- metric prototype for exact match, overlap, top-k hit, false positives, and false negatives;
- six real Aurora classifier calls on DPG-style samples, including African Storybook -> SDG 4 / Quality Education with score `0.951754928`;
- payload-shape finding: Aurora accepted `{"text":"..."}` and failed on direct project metadata fields;
- API base URL deployment patch candidate, now treated as supporting evidence because another contributor has posted a similar deployment/CI direction;
- rate-limit wrapper design note;
- internal proof notes kept in the vault proof packet.

Local proof paths:

- `/Users/rahul/Desktop/CFGT/tasks/chaoss-dpg-10-row-pilot.csv`
- `/Users/rahul/Desktop/CFGT/tasks/chaoss-proof/dpg-metric-demo.md`
- `/Users/rahul/Desktop/CFGT/tasks/chaoss-proof/api-base-url-env-patch.md`
- `/Users/rahul/Desktop/CFGT/tasks/chaoss-proof/rate-limit-wrapper-prototype.md`
- `/Users/rahul/Desktop/CFGT/tasks/chaoss-proof/aurora-single-row-result.json`

These are local proof artifacts for proposal development. Before submission, the best next public proof is probably not another deployment-config note. A stronger proof artifact would be a small benchmark/evaluation package: the 10-row schema, the metric notebook, the single real Aurora result, and the payload-normalization finding. That shows progress on the part of the issue that is hardest to fake: measuring whether the classifier agrees with DPG Registry SDG relevance.

Public posting is intentionally deferred until I have approval to publish. Until then, the proof packet remains local/vault-only.

## Timeline

| Period | Work | Output |
| --- | --- | --- |
| Community bonding / pre-start | Confirm metric, hosting, and sampling method with mentors; prepare public proof links | mentor answers, finalized benchmark schema, public proof comment/PR |
| Week 1 | Setup audit and deployment decision | deployment decision note, environment config patch, CI baseline |
| Week 2 | Benchmark pilot and metric confirmation | reviewed 10-row pilot, metric notebook, confirmed accuracy definition |
| Weeks 3-4 | 100-row benchmark baseline | 100-row spreadsheet, baseline predictions, mismatch categories |
| Weeks 5-6 | API reliability and output hardening | rate-limit wrapper, retries/errors, JSON verification, YAML if required |
| Weeks 7-8 | Classifier/preprocessing improvements | threshold/preprocessing changes tied to benchmark failure modes |
| Weeks 9-10 | Public deployment and CI/CD hardening | public MVP URL, build/deploy workflow, deployment docs |
| Weeks 11-12 | Final benchmark rerun and handoff | before/after report, notebook, maintainer docs, final demo script |

## Testing And Evaluation

| Test Area | Method |
| --- | --- |
| Frontend build | `npm run build` |
| Frontend lint/type safety | lint/type checks, including API client config |
| Backend syntax/tests | import checks for Flask app/provider modules, request-normalizer tests, provider-wrapper tests, and route smoke tests |
| API rate limiting | unit tests or harness proving one request per second per provider |
| Benchmark metrics | notebook assertions for exact, overlap, top-k, false positive, false negative |
| Output export | JSON/YAML structure check with sample result |
| Deployment | public frontend can reach configured backend; local fallback still works |
| Accuracy | baseline vs final benchmark comparison |

Success criteria:

- public tool is accessible without paid hosting;
- CI/CD runs on relevant changes;
- 100 DPG Registry rows are evaluated;
- the selected accuracy metric reaches the 85% target; if it misses, the final report will show the measured gap, failure categories, and concrete remediation path;
- mismatches are explained rather than hidden;
- maintainers can rerun the benchmark and inspect outputs.

## Risks And Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| GitHub Pages cannot host Flask | public MVP blocked if frontend/backend are treated as one app | split deployment; decide backend host early |
| Mentors prefer exact-set accuracy instead of the default Jaccard metric | measured score may be lower | keep exact-set match as a secondary metric from day one and report the gap plainly |
| Aurora/OSDG rate limits slow benchmark | benchmark unreliable or blocked | queue calls, cache outputs, retry safely |
| OSDG token unavailable | OSDG route cannot be fully tested | keep token-dependent path documented; use available classifiers for baseline |
| ML dependencies too heavy for selected host | backend deployment fails | host audit in Week 1; document fallback |
| Existing dependency vulnerabilities | production-readiness concern | audit and fix/triage as hardening work |
| DPG labels are broad or subjective | mismatch report may look worse than tool quality | include mismatch categories and mentor-reviewed relevance option |

## Mentor Questions

1. I propose Jaccard overlap `>= 0.5` as the primary 85% accuracy metric, with exact-set match and top-k hit rate reported separately. Do you prefer a stricter metric?
2. I propose using the first 100 DPG Registry projects with accessible README/About/registry text and recording exclusions. Would you prefer random or stratified sampling?
3. I propose GitHub Pages for the frontend and PythonAnywhere free hosting for the Flask backend if dependency limits fit, with Vercel as fallback. Is that acceptable?
4. I plan to lock the JSON schema first. Would YAML be useful as a second export format, or should I keep the output surface JSON-only?

I will fold mentor answers into the final scope, timeline, and evaluation method before submission.

## Availability

I can commit the expected DMP workload across the coding period. I will front-load setup, benchmark design, and mentor alignment in the first two weeks because those decisions control the rest of the project.

Exact weekly hours and any academic conflicts: I can commit about 20 hours per week during the DMP period, with extra time front-loaded for setup, benchmark design, and mentor alignment.

## Motivation

This project is interesting because it sits at the boundary between open-source infrastructure and evaluation. A classifier is only useful if people can run it, trust its outputs, and understand when it is wrong. The DPG Registry benchmark gives this project a practical way to improve: inspect real cases, measure mismatch, and make targeted changes.

I want to work on this because it is not a vague AI project. The acceptance criteria are concrete: deploy it, benchmark it, improve it, and document it. That is the kind of scope where I can show progress every week.

## References

- CHAOSS issue #8: https://github.com/chaoss/UNSDG-classifier-tool/issues/8
- UN-SDG classifier repository: https://github.com/chaoss/UNSDG-classifier-tool
- DPG Registry: https://www.digitalpublicgoods.net/registry
- C4GT sample proposal: https://github.com/Code4GovTech/C4GT/wiki/Sample-Proposal
- C4GT evaluation criteria: https://github.com/Code4GovTech/C4GT/wiki/Evaluation-Criteria
- GitHub Pages static hosting docs: https://docs.github.com/pages/getting-started-with-github-pages/what-is-github-pages
- PythonAnywhere free account features: https://help.pythonanywhere.com/pages/FreeAccountsFeatures/
- PythonAnywhere Flask setup docs: https://help.pythonanywhere.com/pages/Flask
- Vercel Flask docs: https://vercel.com/docs/frameworks/backend/flask
