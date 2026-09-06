---
name: coolify-deployment-planning
description: 'Plan a Coolify deployment for this FastAPI and Vite/React application. Use before deployment, domain changes, frontend API configuration, or production authentication setup to prevent frontend-to-backend routing mismatches.'
argument-hint: 'Provide the planned frontend URL, backend URL, Git commit or branch, and whether this is a fresh or existing deployment.'
---

# Coolify Deployment Planning

## Scope

Use this skill before deploying or changing the production topology of this repository. The deployment has separate resources for:

- PostgreSQL
- FastAPI backend from `backend/`
- Vite/React frontend served by Nginx from `frontend/`

The frontend origin is not the backend origin. Treat them as different services unless an explicit reverse proxy is configured and verified.

## Source and Routing Contract

Before deployment, inspect the current source at the exact commit to be deployed:

- `frontend/src/api/client.js`
- `frontend/src/context/AuthContext.jsx`
- `frontend/Dockerfile`
- `frontend/nginx.conf`
- `frontend/vite.config.js`
- `backend/app/main.py`
- `backend/app/api/auth.py`

Confirm all browser API calls use the shared API client or another single configured base URL. Search for direct `axios` imports and hardcoded calls such as `axios.post('/api/...')`; these can bypass `VITE_API_BASE_URL` and send API requests to the static frontend host.

Confirm the production values:

- Frontend domain serves the built SPA on port `80`.
- Backend domain serves FastAPI on port `8000`.
- Frontend build variable is `VITE_API_BASE_URL=https://<backend-domain>/api`.
- Backend `CORS_ORIGINS` contains the exact HTTPS frontend origin, without a trailing-path mismatch.
- Nginx either intentionally does not proxy `/api`, in which case the frontend must use the backend origin, or has a verified `/api` reverse proxy.

Do not assume the Vite development proxy applies to the production Nginx image. `vite.config.js` only affects local development.

## API Contract Preflight

Before deployment, obtain the backend OpenAPI definition or Swagger UI and verify the login contract:

- Method is `POST`.
- Route is `/api/auth/token`.
- Request content type is `application/x-www-form-urlencoded`.
- Required fields are `username` and `password`.

Use the backend URL for this check, not the frontend URL. A `405` from the frontend domain does not disprove the backend route.

## Build-Time Verification

Build the frontend from its own directory with the production API variable set. Inspect the generated bundle or a browser request to confirm:

- The configured backend origin is present where expected.
- Authentication calls do not contain a hardcoded frontend-relative `/api/auth/token` unless Nginx explicitly proxies it.
- The deployed commit reported by Coolify matches the source commit that was inspected.

Keep production secrets out of Vite variables. Only public, non-secret API configuration belongs in `VITE_*` build variables.

## Required Acceptance Checks

Before declaring the plan ready:

1. `GET https://<backend-domain>/health` returns `200`.
2. `POST https://<backend-domain>/api/auth/token` with dummy credentials reaches FastAPI and returns `401`, not `404` or `405`.
3. CORS preflight from the exact frontend origin returns `200` and allows `POST` and the required content type.
4. The frontend bundle is built from the intended commit.
5. A browser login test is performed from the deployed frontend, with the Network request host recorded.

Record domains, commit IDs, status codes, and response shapes. Never record tokens, passwords, database URLs, or authorization headers.
