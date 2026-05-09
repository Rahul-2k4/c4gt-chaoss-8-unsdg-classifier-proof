# API Base URL Deployment Patch Candidate

Status: proposal proof note. Not yet submitted as a public PR.

## Current Blocker

The frontend API client currently hardcodes the Flask backend URL:

```ts
const API_BASE_URL = "http://127.0.0.1:5000/";
```

File:

`/Users/rahul/Desktop/CFGT/.tmp/UNSDG-classifier-tool/frontend/services/api.ts`

This works locally but blocks public deployment. A static frontend on GitHub Pages, Vercel, or another no-cost host cannot call `127.0.0.1` on the user's machine and reach the deployed Flask backend.

## Concrete Patch

```ts
const API_BASE_URL = process.env.NEXT_PUBLIC_API_BASE_URL || "http://127.0.0.1:5000/";
```

The deployment documentation should then define:

```bash
NEXT_PUBLIC_API_BASE_URL=https://<deployed-backend-host>/
```

## README Note Candidate

```md
### Frontend API configuration

For local development, the frontend defaults to `http://127.0.0.1:5000/`.
For deployment, set:

NEXT_PUBLIC_API_BASE_URL=https://<your-backend-host>/
```

## Proposal Implication

This is a small but concrete MVP-readiness issue. It proves that deployment is more than a hosting choice. The frontend/backend contract needs production configuration before the tool can be public.

## Validation Note

- local frontend keeps working with the default localhost backend;
- deployed frontend can be pointed at a real backend via `NEXT_PUBLIC_API_BASE_URL`;
- no code path should require hardcoded localhost in production.
