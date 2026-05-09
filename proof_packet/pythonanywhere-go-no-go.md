# PythonAnywhere Go/No-Go

Status: provisional no-go until runtime evidence is collected.

## Imports

- Go if the Flask app imports cleanly on the target free plan without extra system packages.
- Current local proof shows the app can run locally with runtime env overrides, but that is not the same as a PythonAnywhere import/runtime pass.

## Dependencies

- Go if the required dependency set fits the free-plan install/runtime constraints.
- No-go if Torch, sentence-transformers, or any transitive package needs unsupported system libraries or exceeds the free plan budget.

## Runtime Limits

- Go if classifier routes can serve repeated requests without temp/cache instability.
- No-go if the free plan cannot sustain the model load, request size, or CORS split needed for the frontend/backend setup.

## Decision

- Current decision: no-go pending a real PythonAnywhere import and route smoke test.
- Re-evaluate after a clean import, dependency install, and one classifier request succeed on the host.
