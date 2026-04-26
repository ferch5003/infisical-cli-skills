---
name: infisical-tokens
description: Infisical token management - service tokens and token renewal. Use when user mentions infisical token, infisical service-token, service token, renew token.
metadata:
  openclaw:
    requires:
      bins: [infisical]
      credentials:
        - name: INFISICAL_TOKEN
          description: Infisical token for authentication
          required: true
    network:
      - description: Outbound HTTPS to Infisical API
        scope: token creation and management
---

# infisical-tokens

Token management for Infisical CLI — service tokens and access token renewal.

## Commands

| Command | Description |
|---------|-------------|
| `infisical service-token create` | Create a new service token for CI/CD |
| `infisical token renew` | Renew a universal auth access token |

**Note:** Service token **list** and **delete** operations are not available via CLI — use the Infisical web UI. Personal access tokens (PATs) are also managed via the web UI.

## service-token create

Create a service token for machine-to-machine authentication (CI/CD, servers, etc.).

```bash
infisical service-token create [flags]
```

**Flags:**

| Flag | Description | Default |
|------|-------------|---------|
| `--name` (`-n`) | Token name/label | "Service token generated via CLI" |
| `--projectId` | Project ID scope | auto (from `.infisical.json`) |
| `--access-level` (`-a`) | Access level: `read`, `write`, or both | required |
| `--expiry-seconds` (`-e`) | Expiration in seconds from now (0 = never) | `86400` (1 day) |
| `--scope` (`-s`) | Env + path scope: `<env>:<path>` | all envs, root |
| `--token-only` | Print only the token (no metadata) | `false` |

### Access Levels

```bash
# Read-only
infisical service-token create --access-level read

# Read + write
infisical service-token create --access-level read --access-level write
```

### Scope Format

The `--scope` flag uses `<environment>:<folder-path>` format:

```bash
# Production environment, root path
infisical service-token create --scope production:/

# Development environment, specific folder
infisical service-token create --scope development:/backend

# Multiple scopes
infisical service-token create --scope production:/ --scope staging:/api
```

### Examples

**Basic CI/CD token (1 day expiry, read-only):**
```bash
infisical service-token create \
  --name "GitHub Actions CI" \
  --access-level read \
  --expiry-seconds 86400
```

**Long-lived production token (read + write):**
```bash
infisical service-token create \
  --name "Production Deploy" \
  --access-level read \
  --access-level write \
  --scope production:/ \
  --expiry-seconds 31536000
```

**Scoped to specific environment and path:**
```bash
infisical service-token create \
  --name "Backend Dev" \
  --access-level read \
  --scope development:/backend \
  --expiry-seconds 604800
```

**Token-only output (for scripting):**
```bash
# Get token and use it immediately
export INFISICAL_TOKEN=$(infisical service-token create \
  --name "CI Token" \
  --access-level read \
  --expiry-seconds 86400 \
  --token-only)
echo $INFISICAL_TOKEN
```

**With explicit project ID:**
```bash
infisical service-token create \
  --name "Deploy Token" \
  --projectId=proj_xxxxxxxxxxxx \
  --access-level read \
  --access-level write \
  --expiry-seconds 2592000
```

### Expiry Reference

| Seconds | Duration |
|---------|----------|
| `3600` | 1 hour |
| `86400` | 1 day |
| `604800` | 1 week |
| `2592000` | 30 days |
| `31536000` | 1 year |
| `0` | Never expires |

## token renew

Renew a universal auth access token.

```bash
infisical token renew <access-token>
```

**Arguments:**
- `<access-token>` — The access token to renew

```bash
# Renew an access token
infisical token renew eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Personal Access Tokens (PATs)

Personal access tokens are **managed via the Infisical web UI**, not via CLI.

To create or manage PATs:
1. Go to [app.infisical.com](https://app.infisical.com)
2. Navigate to **Settings → Access Tokens**
3. Create, view, or revoke tokens from there

## Service Token vs Universal Auth

| Use Case | Method |
|----------|--------|
| CI/CD pipelines | `infisical service-token create` |
| Machine identity (K8s, GCP, AWS) | `infisical login --method <type>` |
| Renew universal auth token | `infisical token renew` |
| Personal access (human) | Web UI only |

## Related Commands

| Command | Description |
|---------|-------------|
| `infisical login` | Authenticate with Infisical |
| `infisical run` | Run with secrets injected (uses tokens) |
| `infisical secrets` | Manage secrets (uses tokens) |