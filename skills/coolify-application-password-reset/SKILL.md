---
name: coolify-application-password-reset
description: "Reset an application user's password in the production database hosted on Coolify. Use when the local database was changed by mistake, a Coolify backend account reset is needed, or production login does not accept the new password."
---

# Coolify Application Password Reset

## Scope

Use this workflow for the `aimy-security-scanning` production deployment. The target resources are:

- Project: `aimy-security-scanning`
- Environment: `production`
- Backend application: `aimy-scanner-backend`
- Database: `aimy-scanner-postgres`

Reset the account through the backend application terminal. This uses the deployed application code, password hashing configuration, and production `DATABASE_URL` without exposing database credentials.

## Preconditions

1. Confirm the target is the Coolify backend terminal, not a local PowerShell terminal.
2. Confirm the terminal belongs to `aimy-scanner-backend` in the `production` environment.
3. Use a new password that has not appeared in chat, shell history, logs, URLs, or command arguments.
4. Do not retrieve or print `DATABASE_URL`, `SECRET_KEY`, password hashes, access tokens, or database passwords.

## Reset Command

Run this one line in the Coolify backend application terminal:

```sh
python -c "import getpass;from app.database import SessionLocal;from app.models import User;from app.auth import hash_password;p=getpass.getpass('New password: ');d=SessionLocal();u=d.query(User).filter(User.username=='admin').first();assert u,'admin user not found';u.hashed_password=hash_password(p);u.is_active=True;d.commit();d.close();print('production admin account reset')"
```

The value passed to `getpass()` is only a prompt label. The password must be typed interactively after `New password:` appears. Never replace the prompt label with the password and never put the password in the command.

Use `python3` if the container does not provide `python`.

## Verification

1. Require the terminal output `production admin account reset`.
2. Test login at the production frontend in a private browser window or after removing the site's stale `token` from browser storage.
3. The deployed frontend should send authentication to:

   `https://rl5gu6zvdwdkcsmpcyj4geko.coolify.aimy.flairstech.com/api`

4. If login still fails, verify the request returns `401` from that backend and confirm the browser is not using a stale token or a different API host.
5. If the command cannot import `app`, the command is running outside the backend container; stop and reopen the Coolify application terminal.

## Incident Handling

Any password included in a command, prompt label, shell history, screenshot, log, or chat must be treated as exposed. Do not reuse it. Reset again with a newly generated password entered only at the interactive prompt, and rotate any other credential that was exposed alongside it.
