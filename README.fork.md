# Fork Notes

This is a private fork of [chatwoot/chatwoot](https://github.com/chatwoot/chatwoot).

## CI/CD Pipeline

Every merge to `develop` triggers a CircleCI pipeline that:

1. Runs the full test suite (lint, frontend, backend)
2. Builds a Docker image and pushes it to **GitHub Container Registry (GHCR)**
3. SSHs into the **DigitalOcean** droplet and deploys

### Required CircleCI Environment Variables

Set these in **CircleCI → Project Settings → Environment Variables**:

| Variable | Description |
|---|---|
| `GHCR_TOKEN` | GitHub PAT with `write:packages` + `read:packages` scopes |
| `DO_SSH_KEY` | Raw PEM content of the droplet's SSH private key |
| `DO_HOST` | Droplet IP or hostname |
| `APP_ENV` | Base64-encoded production `.env` — run `base64 -i .env` to generate |

### First-time Server Setup

```bash
# On the droplet
mkdir -p /opt/chatwoot
```

Docker must already be installed on the droplet.

### Image Tags

Each deploy pushes two tags to `ghcr.io/<org>/chatwoot`:

- `:develop` — always points to the latest develop build
- `:sha-<commit>` — immutable, pinned to a specific commit

## Syncing Upstream

```bash
git fetch upstream
git merge upstream/develop
```

`.circleci/config.yml` is protected via `.gitattributes` (`merge=ours`) — git keeps our version automatically. To intentionally pull upstream CI changes:

```bash
git checkout upstream/develop -- .circleci/config.yml
# re-add the deploy job and workflow entry at the bottom
```
