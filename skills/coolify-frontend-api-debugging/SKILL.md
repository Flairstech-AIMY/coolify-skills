---
name: coolify-frontend-api-debugging
description: 'Debug frontend-to-backend API failures in Coolify, especially 405, 404, CORS, unauthorized, stale-bundle, or login errors. Use when a deployed browser request fails or the frontend and API have separate domains.'
argument-hint: 'Provide the failing URL, HTTP method, status code, frontend URL, backend URL, and any browser Network or console evidence.'
---

# Coolify Frontend API Debugging

## Evidence First

Start with the exact browser Network record. Capture only non-secret facts:

- Request URL and hostname
- HTTP method
- Status code
- Request content type
- Response content type and bounded body preview
- `Origin`, `Access-Control-Allow-Origin`, and preflight status when relevant
- The loaded frontend asset URL and deployed commit if available

Do not classify a failure as an authentication problem until the method, host, route, and response status are checked. A `405 Method Not Allowed` usually indicates that the request reached the wrong service or a route that does not accept that method; it is not an invalid-credentials response.

## Classify by Host

For this deployment, compare the request host with the resource owning it:

| Request host | Expected behavior |
| --- | --- |
| Frontend domain | Static SPA/Nginx responses; may return `405` for API POSTs when no proxy exists |
| Backend domain | FastAPI routes, including `/api/auth/token` |

If the request is sent to the frontend host, inspect the built JavaScript rather than changing backend auth code. Search the deployed asset for the login call and determine whether it uses a relative `/api` URL or the configured backend origin.

## Contract and Direct Probes

Read the live backend `/docs` or `/openapi.json` before changing code. For the token route, send a bounded test with dummy credentials:

```powershell
Invoke-WebRequest `
  -Uri 'https://<backend-domain>/api/auth/token' `
  -Method Post `
  -Body @{ username = 'probe'; password = 'probe' } `
  -ContentType 'application/x-www-form-urlencoded'
```

Expected result for dummy credentials is `401 Incorrect username or password`. This proves the request reached the backend route without testing or exposing a real account.

Then test CORS from the exact frontend origin with an `OPTIONS` preflight. Distinguish:

- `401` from the backend: route and method work; credentials or account may be the issue.
- `405` from the frontend host: wrong host or missing reverse proxy.
- CORS browser error with a successful direct backend response: origin, allowed headers, or preflight configuration.
- `404` from backend: wrong API prefix, deployment version, or route registration.
- Request still shows an old asset or old path: browser/CDN cache or stale deployment.

## Deployment and Cache Checks

Verify all of the following before retrying a login:

1. Coolify resource UUID and domain mapping.
2. Frontend and backend deployed commits.
3. Frontend production build variable key and scope, without exposing secrets.
4. Current HTML asset filename and the API call inside that asset.
5. Browser cache, service worker, or stale tab state.

Do not infer the running code from the local working tree alone. The deployed asset and Coolify deployment commit are the source of truth for a production browser failure.

## Reporting

State the confirmed boundary, one discriminating check, and the next action. Example: `POST` to the frontend Nginx domain returns `405`; the same form request to the backend domain returns `401`; therefore the API is healthy and the frontend bundle or proxy routing is wrong. Redact passwords, tokens, cookies, authorization headers, and full secret-bearing environment values.
