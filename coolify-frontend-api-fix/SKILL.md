---
name: coolify-frontend-api-fix
description: 'Fix and verify frontend-to-backend API routing failures in this Coolify deployment. Use after diagnosing a 405, wrong-host login request, stale frontend bundle, missing API client usage, or related CORS configuration issue.'
argument-hint: 'Provide the failing request, frontend/backend domains, current deployed commit, and whether code, proxy, or environment configuration is being changed.'
---

# Coolify Frontend API Fix

## Choose the Smallest Correct Fix

Use the diagnosed boundary to select one fix:

- If a component imports raw `axios` or uses a hardcoded `/api/...` path, route it through `frontend/src/api/client.js` and use the shared base URL.
- If the intended architecture is same-origin `/api`, add and verify an Nginx reverse proxy to the backend; do not add it implicitly to a static frontend container.
- If the frontend and backend have separate HTTPS domains, set the production `VITE_API_BASE_URL` to `https://<backend-domain>/api` as a build-time variable.
- If the backend receives the request but rejects the origin, set production `CORS_ORIGINS` to the exact frontend origin and redeploy the backend.
- If source is correct but the browser asset is old, deploy the intended commit and invalidate stale browser or CDN state.

Do not weaken authentication, allow all origins with credentials, expose backend secrets to Vite, or change a `405` into a permissive fallback route.

## Implementation Checks

For a code fix:

1. Keep login request data as `application/x-www-form-urlencoded` with `username` and `password`.
2. Preserve the backend route `/api/auth/token` behind the configured client base URL.
3. Search the frontend for other direct API calls that bypass the shared client.
4. Build from `frontend/`, not the repository root.
5. Run source diagnostics and `npm run build`.

For a deployment-only fix:

1. Confirm Coolify is deploying the current Git commit, not only the current local files.
2. Confirm the production, not preview, `VITE_API_BASE_URL` value.
3. Redeploy the frontend after changing a Vite build variable or source code.
4. Wait for deployment status `finished` and record the commit.

## End-to-End Verification

Verify in this order:

1. Backend `GET /health` returns `200`.
2. Backend `POST /api/auth/token` with dummy form credentials returns `401`, proving route reachability.
3. Backend CORS preflight from the frontend origin returns `200` and allows `POST`.
4. Frontend HTML references the new asset after redeployment.
5. The deployed JavaScript contains the configured backend origin or the approved proxy path.
6. A real browser login request goes to the backend host and returns the expected success or credential error, never the frontend Nginx host.
7. After login, reload a protected route and verify token injection and logout behavior on `401`.

A direct `POST` to the frontend hostname may still return `405` when Nginx is intentionally static-only. That is acceptable if browser API calls use the backend hostname and the backend route passes the checks above.

## Closeout

Report the changed file or configuration, deployed commit, deployment status, exact verification URLs, and observed status codes. Do not include credentials, access tokens, cookies, database URLs, or authorization headers. Preserve failed deployment records and do not delete or recreate the database while fixing frontend routing.
