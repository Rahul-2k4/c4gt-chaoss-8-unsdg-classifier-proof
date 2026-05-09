# API URL PR Draft

Status: internal draft only. Do not post without Rahul approval.

## Problem

The frontend currently hardcodes the backend API URL:

```ts
const API_BASE_URL = "http://127.0.0.1:5000/";
```

This works for local development but breaks public deployment. A hosted frontend cannot call the user's localhost and reach the Flask backend.

## Proposed Patch

File:

`frontend/services/api.ts`

```diff
- const API_BASE_URL = "http://127.0.0.1:5000/";
+ const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL || "http://127.0.0.1:5000/";
```

## README Note

```md
### Frontend API configuration

For local development, the frontend defaults to `http://127.0.0.1:5000/`.

For deployment, set:

NEXT_PUBLIC_API_BASE_URL=https://<your-backend-host>/
```

## Why This Helps Issue #8

Issue #8 requires public hosting and CI/CD. This patch is the smallest production-readiness change because it separates local development from deployed backend configuration.

## Test Plan

- Run frontend lint.
- Run frontend production build.
- Start local backend at `http://127.0.0.1:5000/` and confirm default still works.
- Set `NEXT_PUBLIC_API_BASE_URL` to a test URL and confirm generated frontend client points to configured backend.

## PR Body Draft

```md
## Summary

- Make frontend backend URL configurable with `NEXT_PUBLIC_API_BASE_URL`.
- Keep `http://127.0.0.1:5000/` as local fallback.
- Document deployment config path.

## Why

The current frontend always calls localhost, which blocks public deployment for the UN-SDG classifier MVP. Issue #8 asks for a hosted MVP, so frontend/backend configuration needs to work outside local development.

## Test Plan

- npm run lint
- npm run build
- local fallback remains `http://127.0.0.1:5000/`
```
