---
name: coolify-terminal-access
description: 'Use when guiding a user through Coolify terminal access for a deployed application or database, including finding the correct resource terminal, inspecting PostgreSQL roles safely, and running bounded server-side commands without exposing secrets.'
argument-hint: 'Provide the Coolify project UUID, environment UUID, resource UUID, and whether the target is an application or database terminal.'
---

# Coolify Terminal Access

## Scope

Use this skill when the user can access a Coolify resource terminal but needs help operating it safely. Prefer the Coolify resource terminal over public database exposure or guessed SSH routes. Keep the operation limited to the named project, environment, and resource.

Do not request or print database passwords, `DATABASE_URL` values, JWT secrets, API tokens, private keys, authorization headers, or plaintext account passwords. Do not use a browser automation tool unless the user explicitly authorizes browser interaction; a user-provided Coolify terminal link is sufficient authorization to guide terminal commands.

## Identify the Correct Terminal

Distinguish the two terminal types:

- **Database terminal:** useful for `psql`, checking database roles, listing application usernames, and executing narrowly scoped SQL.
- **Backend application terminal:** preferred for account recovery because it has the deployed application code, password-hashing implementation, and runtime database configuration.

Construct resource links from the Coolify UI context:

```text
https://<coolify-host>/project/<project_uuid>/environment/<environment_uuid>/database/<database_uuid>/terminal
https://<coolify-host>/project/<project_uuid>/environment/<environment_uuid>/application/<application_uuid>/terminal
```

Confirm the resource name and UUID before giving commands. Never assume a project display name is a database hostname.

## Database Terminal Workflow

Use environment-provided PostgreSQL variables without printing their values:

```sh
printf '%s\n' "$POSTGRES_USER" "$POSTGRES_DB"
psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"
```

If the role is unknown, list only variable names:

```sh
env | awk -F= '/^(POSTGRES|PG)/ { print $1 }'
```

Inside `psql`, inspect only non-sensitive account metadata:

```sql
\conninfo
SELECT username, is_active FROM users ORDER BY id;
```

Do not run plain `env`, select password hashes, or paste connection strings into chat.

## Backend Terminal Workflow

For account recovery, run the reset inside the deployed backend container so the production password algorithm and database configuration are used. Prefer a heredoc that prompts for the password interactively:

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

Use `python3` if `python` is unavailable. Do not put the password in the command, shell history, SQL, deployment variables, or chat. The prompt text is not the password; the password is what the user types after the prompt appears.

## Verification

Verify only a success message from the reset script, then test login through the deployed frontend. If troubleshooting the frontend, clear only the application token from that site’s browser storage:

```js
localStorage.removeItem('token')
```

Record status codes and redacted error categories only. Never report tokens, password hashes, connection details, or plaintext passwords. If the resource terminal cannot reach the intended database or application package, stop and identify the missing access path rather than exposing PostgreSQL publicly or recreating the database.
