---
name: infisical-cli
description: Comprehensive Infisical CLI command reference and workflows for secrets management and machine identity. Use when user mentions Infisical CLI, infisical commands, secrets management, secrets injection, environment variables from secrets manager, or any Infisical terminal operations. Triggers on infisical, Infisical CLI, infisical login, infisical secrets, infisical run, infisical init, secrets injection, dynamic secrets.
metadata:
  openclaw:
    requires:
      bins: [infisical]
      install:
        - id: brew
          kind: brew
          formula: infisical
          bins: [infisical]
          label: Install infisical (brew)
        - id: download
          kind: download
          url: https://github.com/Infisical/infisical/releases
          label: Download infisical binary
    credentials:
      - name: INFISICAL_TOKEN
        description: Infisical personal access token or service token for API access. Used by automation scripts to fetch secrets, run commands with injected secrets, and manage machine identities. If not set, falls back to infisical CLI config (~/.config/infisical/config.json).
        required: false
        fallback: infisical config (set via infisical login or service-token)
    network:
      - description: Outbound HTTPS to your Infisical instance (default US Cloud app.infisical.com). HTTPS enforced; token never sent over HTTP.
        scope: authenticated API calls for secrets, projects, identities
    write_access:
      - description: Scripts can create, update, and delete secrets, manage projects, identities, and configure secrets injection. Review scripts before use in automated or agentic contexts.
---

# Infisical CLI Skills

Comprehensive Infisical CLI command reference for secrets management and machine identity.

## Quick start

```bash
# First time setup - authenticate with browser
infisical login

# Initialize a project (creates .infisical.json) — interactive
infisical init

# View secrets
infisical secrets                      # List all secrets
infisical secrets --env=development     # List secrets for specific environment

# Run command with secrets injected
infisical run --env=production -- node app.js

# Dynamic secrets (lease temporary credentials)
infisical dynamic-secrets lease create db-creds --env=production
```

## Multi-domain configuration

### US Cloud (default)
```bash
infisical config set API_URL https://api.infisical.com
```

### EU Cloud
```bash
infisical config set API_URL https://eu-api.infisical.com
```

### Self-hosted
```bash
infisical config set API_URL https://your-instance.com/api
infisical config set INFISICAL_URL https://your-instance.com
```

## Authentication methods

### Browser login (interactive)
```bash
infisical login
infisical login --email user@example.com
```

### Universal Auth (machine-to-machine)
```bash
infisical login --method universal-auth \
  --client-id <id> --client-secret <secret>
```

### Machine identity auth

| Provider | Command |
|----------|---------|
| Kubernetes | `infisical login --method kubernetes --machine-identity-id <id> --service-account-token-path <path>` |
| Azure | `infisical login --method azure --machine-identity-id <id> --client-id <azure-id> --tenant-id <tenant>` |
| GCP (IAM) | `infisical login --method gcp-iam --machine-identity-id <id> --service-account-key-file-path <path>` |
| GCP (ID Token) | `infisical login --method gcp-id-token --jwt <token>` |
| AWS | `infisical login --method aws-iam --machine-identity-id <id> --role-arn <arn>` |
| OIDC | `infisical login --method oidc-auth --jwt <token>` |
| JWT | `infisical login --method jwt-auth --jwt <token>` |

## Environment variables

| Variable | Description |
|----------|-------------|
| `INFISICAL_TOKEN` | Personal/service token (bypasses login) |
| `INFISICAL_API_URL` | API endpoint (default: https://app.infisical.com/api) |
| `INFISICAL_DISABLE_UPDATE_CHECK` | Set to 1 to disable update check |

## Skill organization

**Core sub-skills:**

| Skill | Domain |
|-------|--------|
| `infisical-auth` | Login, logout, reset, user management |
| `infisical-secrets` | CRUD operations for secrets |
| `infisical-dynamic-secrets` | Dynamic secrets via lease model |
| `infisical-run` | Command execution with secrets injection |
| `infisical-init` | Project initialization (interactive) |
| `infisical-bootstrap` | Self-hosted instance bootstrap |
| `infisical-export` | Export secrets to file |
| `infisical-tokens` | Service tokens and token renewal |

**Infrastructure sub-skills:**

| Skill | Domain |
|-------|--------|
| `infisical-ssh` | SSH certificate management |
| `infisical-pam` | Privileged access management |
| `infisical-kmip` | KMIP server operations |
| `infisical-relay` | Relay server management |
| `infisical-gateway` | API gateway configuration |
| `infisical-vaults` | Vault integration |

## Command reference

| Command | Description |
|---------|-------------|
| `infisical login` | Authenticate with Infisical |
| `infisical init` | Initialize project (interactive) |
| `infisical secrets` | Manage secrets (get, set, delete) |
| `infisical secrets folders` | Manage folder structure |
| `infisical secrets generate-example-env` | Export .env file |
| `infisical dynamic-secrets lease create` | Lease dynamic credentials |
| `infisical dynamic-secrets lease list` | List active leases |
| `infisical dynamic-secrets lease renew` | Renew a lease |
| `infisical dynamic-secrets lease delete` | Revoke a lease |
| `infisical run` | Run command with injected secrets |
| `infisical export` | Export secrets to file |
| `infisical service-token create` | Create service token |
| `infisical token renew` | Renew access token |
| `infisical ssh` | Manage SSH certificates |
| `infisical reset` | Clear local configuration |
| `infisical user get` | Show current user info |
| `infisical scan` | Scan for leaked secrets |
| `infisical bootstrap` | Bootstrap self-hosted instance |

## Common workflows

### Daily development

```bash
infisical login
infisical init
infisical secrets set API_KEY=sk-xxx --env=production
infisical run --env=development -- nodemon src/index.js
```

### CI/CD integration

```bash
export INFISICAL_TOKEN=<service-token>
infisical run --env=production -- npm test
```

### Export secrets to file

```bash
# Correct way (not infisical secrets --export)
infisical export --env=production --format=dotenv > .env
infisical export --env=production --format=json > secrets.json
infisical export --env=production --format=csv > secrets.csv
```

### Dynamic secrets lifecycle

```bash
# Lease credentials
infisical dynamic-secrets lease create db-creds --ttl=1h --env=production

# List leases
infisical dynamic-secrets lease list db-creds --env=production

# Renew (if needed)
infisical dynamic-secrets lease renew <lease-id> --ttl=1h

# Revoke when done
infisical dynamic-secrets lease delete db-creds --env=production
```

## Decision Trees

### "Which auth method?"

```
├─ Local dev → infisical login (browser)
├─ CI/CD → service token (infisical service-token create)
├─ K8s → --method kubernetes --machine-identity-id
├─ GCP → --method gcp-iam or gcp-id-token
├─ AWS → --method aws-iam --machine-identity-id
├─ Azure → --method azure
├─ OIDC → --method oidc-auth --jwt
└─ Universal Auth → --method universal-auth --client-id --client-secret
```

### "How to inject secrets?"

```
├─ Run command → infisical run -- node app.js
├─ Export .env → infisical export --format=dotenv > .env
├─ Single secret → infisical secrets get KEY
└─ Use in script → source <(infisical export --format=dotenv)
```

### "Need dynamic DB credentials?"

```
1. Configure dynamic secret in Infisical web UI (PostgreSQL, MySQL, Redis, AWS RDS, etc.)
2. Lease it → infisical dynamic-secrets lease create <name> --ttl=1h
3. Use credentials in app
4. Revoke → infisical dynamic-secrets lease delete <name>
```

## Security considerations

- Protect `~/.config/infisical/config.json` — contains tokens
- Use minimal scopes for service tokens
- Use `--ttl` on dynamic secrets leases (short-lived credentials)
- Review Infisical audit logs for secret access
- Rotate tokens regularly