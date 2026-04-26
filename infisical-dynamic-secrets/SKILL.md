---
name: infisical-dynamic-secrets
description: Dynamic secrets generation for databases and cloud providers. Use when user mentions dynamic-secrets, dynamic secrets, temporary credentials, database credentials, rotating secrets, lease secrets.
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
        scope: dynamic secrets lease operations
---

# infisical-dynamic-secrets

Manage dynamic secrets using a **lease-based model**. Dynamic secrets are configured in the Infisical dashboard first, then the CLI is used to lease (generate temporary credentials), list leases, renew them, and revoke (delete) them.

**Important:** Providers are configured in the Infisical web UI, not via CLI flags. The CLI only manages the lease lifecycle.

## Command Structure

```
infisical dynamic-secrets lease create   # Lease (generate temp credentials)
infisical dynamic-secrets lease list       # List active leases
infisical dynamic-secrets lease renew     # Renew a lease
infisical dynamic-secrets lease delete   # Revoke/delete a lease
```

## Quick Examples

```bash
# Lease (create temp credentials for a dynamic secret)
infisical dynamic-secrets lease create my-postgres-secret --env=production

# List all leases for a dynamic secret
infisical dynamic-secrets lease list my-postgres-secret --env=production

# Renew a lease (extend its lifetime)
infisical dynamic-secrets lease renew <lease-id> --ttl=1h

# Revoke/delete a lease (invalidate credentials immediately)
infisical dynamic-secrets lease delete my-postgres-secret --env=production
```

## Global Flags

These apply to all `infisical dynamic-secrets lease` subcommands.

| Flag | Description | Default |
|------|-------------|---------|
| `--env <env>` | Environment name | `dev` |
| `--path <path>` | Folder path within project | `/` |
| `--projectId <id>` | Project ID (machine identity auth) | auto |
| `--project-slug <slug>` | Project slug | auto |
| `--token <token>` | Service/machine identity token | auto |

## lease create

Lease a dynamic secret — generates temporary credentials.

```bash
infisical dynamic-secrets lease create <dynamic-secret-name> [flags]
```

**Arguments:**
- `<dynamic-secret-name>` — Name of the dynamic secret (configured in UI)

**Flags:**

| Flag | Description | Default |
|------|-------------|---------|
| `--ttl <duration>` | Lease lifetime (e.g., `1h`, `30m`, `24h`) | dynamic secret default |
| `--output <format>` | Output format: `yaml`, `json`, `dotenv` | — |
| `--plain` | Print credentials without formatting | `false` |
| `--kubernetes-namespace <ns>` | K8s namespace (Kubernetes secrets only) | — |

**Examples:**
```bash
# Lease with default TTL
infisical dynamic-secrets lease create postgres-creds --env=production

# Lease with 1 hour TTL
infisical dynamic-secrets lease create postgres-creds --ttl=1h --env=production

# Lease with 30 minute TTL, JSON output
infisical dynamic-secrets lease create redis-creds --ttl=30m --output=json --env=prod

# Plain output for scripting
infisical dynamic-secrets lease create api-creds --plain --env=production
```

## lease list

List all active leases for a dynamic secret.

```bash
infisical dynamic-secrets lease list <dynamic-secret-name> [flags]
```

**Examples:**
```bash
infisical dynamic-secrets lease list postgres-creds --env=production
infisical dynamic-secrets lease list postgres-creds --env=staging --output=json
```

## lease renew

Renew an existing lease by its ID, extending its lifetime.

```bash
infisical dynamic-secrets lease renew <lease-id> [flags]
```

**Flags:**

| Flag | Description | Default |
|------|-------------|---------|
| `--ttl <duration>` | New lease lifetime | dynamic secret default |

**Examples:**
```bash
# Renew for 1 hour
infisical dynamic-secrets lease renew lease-abc123 --ttl=1h

# Renew for 30 minutes
infisical dynamic-secrets lease renew lease-abc123 --ttl=30m
```

## lease delete

Delete/revoke a lease, immediately invalidating the credentials.

```bash
infisical dynamic-secrets lease delete <dynamic-secret-name> [flags]
```

**Examples:**
```bash
# Revoke credentials immediately
infisical dynamic-secrets lease delete postgres-creds --env=production

# Revoke from specific project
infisical dynamic-secrets lease delete redis-creds --env=prod --projectId=proj_xxx
```

## TTL Format

TTL strings follow duration format:

| Format | Duration |
|--------|----------|
| `30m` | 30 minutes |
| `1h` | 1 hour |
| `24h` | 24 hours |
| `7d` | 7 days |
| `30d` | 30 days |

## Lease Lifecycle

```
┌─────────────────────────────────────────────┐
│ 1. Configure dynamic secret in Infisical UI  │
│    (PostgreSQL, MySQL, Redis, AWS, etc.)    │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│ 2. infisical dynamic-secrets lease create   │
│    → Get temporary credentials               │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│ 3. Use credentials in your application        │
└──────────────────────┬──────────────────────┘
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
┌──────────────────┐   ┌──────────────────────┐
│ 4. lease renew   │   │ 5. lease delete      │
│ (extend if needed)│   │ (invalidate creds)  │
└──────────────────┘   └──────────────────────┘
```

## Common Patterns

### PostgreSQL with auto-reconnect

```bash
# Create lease
CREDENTIALS=$(infisical dynamic-secrets lease create pg-creds --plain --output=dotenv)
export $CREDENTIALS
psql -h $PG_HOST -U $PG_USER -d $PG_DATABASE

# When done, revoke
infisical dynamic-secrets lease delete pg-creds
```

### CI/CD with short-lived credentials

```bash
# Get credentials for database migration
creds=$(infisical dynamic-secrets lease create db-migration --ttl=5m --plain --output=dotenv --env=production)
export $creds
npm run migrate

# Lease auto-expires or revoke immediately
infisical dynamic-secrets lease delete db-migration --env=production
```

### AWS RDS (via Infisical)

```bash
# Lease AWS RDS credentials
infisical dynamic-secrets lease create prod-rds --ttl=1h --env=production --output=dotenv > .aws-rds.env
source .aws-rds.env
```

## Notes

- **Dynamic secrets must be configured in the Infisical web UI** before using the CLI
- The CLI only manages the **lease lifecycle**, not the provider configuration
- Credentials are **temporary by design** — always use lease/delete instead of long-lived static credentials
- Use **`--plain --output=dotenv`** for easy environment variable sourcing in scripts
- Leases can be **renewed** before expiration to extend credential validity
- **AWS RDS**, **Azure SQL**, and **GCP Cloud SQL** are supported as dynamic secret providers in the UI