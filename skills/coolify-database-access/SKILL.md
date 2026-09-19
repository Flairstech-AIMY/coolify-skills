---
name: coolify-database-access
description: 'Safely access, expose, troubleshoot, and recover credentials for Coolify-managed PostgreSQL databases without exposing secrets or destroying data.'
argument-hint: 'Provide the Coolify database UUID or name, intended access method, and whether the operation is local, private-network, or public.'
---

# Coolify Database Access

## Scope

Use this skill when connecting to a Coolify-managed database, changing its network exposure, recovering an application account, or diagnosing database connectivity. Preserve the existing database resource and data unless destructive recreation is explicitly approved.

Do not use the browser unless the user explicitly authorizes it. Prefer the Coolify REST API for resource metadata and configuration, then use SSH or an approved server-side terminal for SQL execution. The Coolify API does not itself provide arbitrary SQL execution.

Never print or commit database passwords, full `DATABASE_URL` values, JWT secrets, API tokens, SSH private keys, or authorization headers.

## Discovery

1. Resolve the database by UUID or exact name.
2. Read the database resource and record only UUID, type, status, `is_public`, port metadata, server, project, and environment.
3. Inspect application environment-variable names only. Never retrieve or echo secret values.
4. Identify the server connection metadata and confirm an approved SSH or terminal path before attempting SQL.
5. Check backups before schema or account changes.

Do not assume the Coolify resource name is the internal Docker hostname. Use the connection template or the application’s configured connection string through an approved secret-handling path.

## Network Exposure

Keep PostgreSQL private by default. Public exposure requires explicit user authorization and a documented operational need.

Before setting `is_public: true`:

- Confirm the database is not already exposed through another port or proxy.
- Confirm firewall and source-IP restrictions are in place.
- Confirm TLS and certificate requirements for clients.
- Record the previous `is_public` state.
- Prefer a temporary, allowlisted tunnel or SSH port forward instead of a public listener.

After a public setting change, read the resource back and verify `is_public`, public port, port mappings, status, and restart timestamps. Do not claim external reachability when the public port is null or unverified.

## Credential Recovery

For an application account reset:

1. Confirm the target deployment, database, and account name.
2. Generate or receive the replacement password without logging it.
3. Hash the password using the application’s existing password algorithm and runtime environment.
4. Execute the update only through an approved server-side SQL or application-admin path.
5. Verify login with a bounded request and discard the token and password from memory.
6. Redeploy or restart only when required by the application’s connection behavior.

Do not reset credentials by editing committed files, frontend variables, deployment logs, or a database container environment variable. Do not expose a plaintext password in chat or shell history.

### Coolify Resource Terminals

When the user can open a Coolify terminal, prefer the backend application terminal for account recovery. It uses the deployed application code and its configured database connection, avoiding mismatches between local and production password-hashing settings.

Use the database terminal only for narrow inspection or SQL that cannot be performed from the backend terminal. The database role may not be `postgres`; use the resource-provided variables without printing the environment:

```sh
psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"
```

Inside `psql`, inspect account metadata without selecting `hashed_password`:

```sql
SELECT username, is_active FROM users ORDER BY id;
```

For a deployed FastAPI application using this repository's `hash_password()` helper, run the reset in the backend terminal with an interactive password prompt:

```sh
python - <<'PY'
import getpass
from app.database import SessionLocal
from app.models import User
from app.auth import hash_password

password = getpass.getpass("New password: ")
db = SessionLocal()
try:
	user = db.query(User).filter(User.username == "admin").first()
	if user is None:
		user = User(username="admin", hashed_password=hash_password(password), is_active=True)
		db.add(user)
		action = "created"
	else:
		user.hashed_password = hash_password(password)
		user.is_active = True
		action = "reset"
	db.commit()
	print(f"admin account {action}")
finally:
	db.close()
PY
```

Use `python3` if needed. The prompt label is not the password; the user types the password after the prompt. Never put the password in the command, SQL, deployment variables, logs, or chat. If the backend terminal cannot import the application package or connect to the configured database, stop and report the missing access path.

## Verification

Record only status codes, resource UUIDs, timestamps, and redacted error categories. Verify:

- PostgreSQL remains healthy.
- The backend can connect using its existing secret configuration.
- The intended account can authenticate.
- Invalid credentials fail.
- No password, token, or connection string appears in logs or responses.

If server-side access is unavailable, report the exact missing access path and stop. Do not recreate the database or guess connection details.
