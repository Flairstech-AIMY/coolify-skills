---
name: coolify-deployment
description: 'Deploy this FastAPI and Vite/React application to Coolify. Use when configuring Git-based Coolify services, Dockerfiles, domains, PostgreSQL, environment variables, health checks, or deployment verification.'
argument-hint: 'Describe the target domains, Git source, database choice, and whether AWS discovery is required.'
---

# Coolify Deployment

## Scope

Use this skill to plan or implement deployment of this repository as separate Coolify resources:

- A private PostgreSQL resource
- A FastAPI backend from `backend/`
- A production frontend from `frontend/`

Do not commit real credentials, generated passwords, AWS keys, or production domains.

## Procedure

1. Inspect the current repository before editing deployment files:
   - `backend/requirements.txt`
   - `backend/app/main.py`
   - `backend/app/database.py`
   - `frontend/package.json`
   - `frontend/src/api/client.js`
   - `frontend/vite.config.js`
2. Confirm the intended Coolify source and domains. Prefer a Git repository, separate HTTPS frontend/API subdomains, and a private database unless the user specifies otherwise.
3. Configure the backend service:
   - Root directory: `backend`
   - Deployment: Dockerfile
   - Internal port: `8000`
   - Command: `uvicorn app.main:app --host 0.0.0.0 --port 8000`
   - Variables: `DATABASE_URL`, `SECRET_KEY`, `ACCESS_TOKEN_EXPIRE_MINUTES`, `CORS_ORIGINS`
   - Add AWS variables only as encrypted Coolify secrets when discovery requires them.
4. Configure the frontend service:
   - Root directory: `frontend`
   - Deployment: Dockerfile
   - Internal port: `80`
   - Build variable: `VITE_API_BASE_URL=https://<api-domain>/api`
   - Configure the production server to serve `dist` and fall back to `index.html` for client-side routes.
5. Configure PostgreSQL:
   - Keep it private and persistent.
   - Use database name `aimy_scanner`.
   - Pass the Coolify internal connection string to the backend.
6. Deploy in dependency order: PostgreSQL, backend, frontend.
7. After the backend is healthy, bootstrap the first admin through `POST /api/auth/users`, then authenticate through the frontend.

## Verification

- Backend health endpoint responds over HTTPS.
- Frontend root and a client-side deep link both return the application.
- Browser requests from the frontend origin reach the backend without CORS errors.
- Login, token injection, logout-on-401, and protected routes work after a page reload.
- AWS discovery works when credentials are configured, without secrets appearing in logs.
- PostgreSQL data survives a backend redeploy.

## Deployment Safety

Before using an existing database, inspect the startup lifecycle in `backend/app/main.py`. This repository currently performs schema checks and can reset scan tables on detected incompatibility. Take a backup and test against staging before production deployment; prefer explicit migrations before relying on an existing production database.
