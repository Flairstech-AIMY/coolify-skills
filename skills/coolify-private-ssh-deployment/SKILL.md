---
name: coolify-private-ssh-deployment
description: 'Deploy private GitHub repositories to Coolify with an SSH deploy key. Use when GitHub App or public-source checkout fails, when selecting Private Repository with Deploy Key, or when configuring private backend/frontend Dockerfile applications.'
argument-hint: 'Provide the private repository SSH URL, branch, application root, internal port, and the Coolify deploy-key name.'
---

# Coolify Private SSH Deployment

## When to Use

Use this path for a private repository when Coolify cannot reliably access GitHub through a public source or GitHub App integration. The repository URL must use SSH, for example:

```text
git@github.com:ORG/REPOSITORY.git
```

Do not switch a private repository to public merely to make deployment work.

## Key Lifecycle

1. Generate a dedicated Ed25519 key outside the repository:

```powershell
ssh-keygen -t ed25519 -C "coolify-deploy" -f "$env:USERPROFILE\.coolify\project-deploy"
```

2. Add only the `.pub` key to the appropriate GitHub repository as a read-only deploy key.
3. Add the private key to Coolify as a named private key.
4. Keep both files outside the repository and confirm they are ignored if the directory could be scanned by tooling.
5. Never print the private key or include it in logs, screenshots, prompts, or commits.

The Coolify private key and the GitHub public deploy key are a pair. Registering a public key with GitHub without adding the matching private key to Coolify, or vice versa, cannot work.

## Repository Access Check

Verify access without relying on cached local Git credentials. On Windows, Git Credential Manager can make a local `git ls-remote` appear successful even when Coolify has no access. Use the SSH identity explicitly:

```powershell
ssh -i "$env:USERPROFILE\.coolify\project-deploy" -o IdentitiesOnly=yes -T git@github.com
git -c core.sshCommand="ssh -i $env:USERPROFILE\.coolify\project-deploy -o IdentitiesOnly=yes" ls-remote git@github.com:ORG/REPOSITORY.git refs/heads/main
```

Confirm the returned commit matches the intended branch. Do not expose the key contents.

## Coolify Application Settings

Create `Private Git Repository (with Deploy Key)` and select the named key:

| Service | Base directory | Build pack | Internal port | Dockerfile |
| --- | --- | --- | --- | --- |
| Backend | `/backend` | Dockerfile | `8000` | `/Dockerfile` |
| Frontend | `/frontend` | Dockerfile | `80` | `/Dockerfile` |

Use branch `main` unless the deployment target explicitly uses another branch. Confirm the repository URL is SSH and the key name is the expected one before saving.

## Deployment Order

Keep the existing PostgreSQL resource. Configure and deploy the backend first, verify `/health`, then create/configure the frontend with the same SSH key. Set the frontend build variable to the public backend API base, such as `https://api.example.com/api`.

## Common Mistakes

- Testing with cached HTTPS credentials instead of the Coolify SSH identity.
- Adding the private key to GitHub instead of the public key.
- Using a repository URL that is valid for a different Git integration.
- Leaving the base directory at `/` when the Dockerfile and application are under `/backend` or `/frontend`.
- Recreating PostgreSQL while replacing only failed application resources.
