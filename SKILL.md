---
name: infisical-cli
description: Comprehensive Infisical CLI command reference and workflows for secrets management and machine identity. Use when user mentions Infisical CLI, infisical commands, secrets management, secrets injection, environment variables from secrets manager, or any Infisical terminal operations. Triggers on infisical, Infisical CLI, infisical login, infisical secrets, infisical run, infisical init, secrets injection, dynamic secrets.
metadata: {"openclaw": {"requires": {"bins": ["infisical"]}, "install": [{"id": "brew", "kind": "brew", "formula": "infisical", "bins": ["infisical"], "label": "Install infisical (brew)"}, {"id": "download", "kind": "download", "url": "https://github.com/Infisical/infisical/releases", "label": "Download infisical binary"}]}}
requirements:
  binaries:
    - infisical
  notes: |
    Requires Infisical authentication via 'infisical login' (stores token in ~/.config/infisical/config.json).
    Supports multiple deployment domains: US Cloud (app.infisical.com), EU Cloud (eu.infisical.com), and self-hosted.
    Some commands may access sensitive data: secrets, SSH keys, certificates, vault credentials.
    Review authentication methods and script contents before autonomous use.
openclaw:
  requires:
    credentials:
      - name: INFISICAL_TOKEN
        description: >
          Infisical personal access token or service token for API access.
          Used by automation scripts to fetch secrets, run commands with injected
          secrets, and manage machine identities. If not set, scripts fall back to
          reading token from infisical CLI config (~/.config/infisical/config.json).
        required: false
        fallback: infisical config (set via infisical login or service-token)
    network:
      - description: Outbound HTTPS to your Infisical instance (default US Cloud app.infisical.com)
        scope: authenticated API calls for secrets, projects, identities; HTTPS enforced; token never sent over HTTP
    write_access:
      - description: >
          Scripts in this skill can create, update, and delete secrets, manage
          projects, identities, and configure secrets injection. Review
          scripts before use in automated or agentic contexts.
---

# Infisical CLI Skills

Comprehensive Infisical CLI command reference and workflows for secrets management.

## Quick start

```bash
# First time setup - authenticate with browser
infisical login

# Initialize a project (creates .infisical.json)
infisical init

# View and manage secrets
infisical secrets                      # List all secrets
infisical secrets --env=development   # List secrets for specific environment

# Run command with secrets injected
infisical run --env=production -- node app.js

# Dynamic secrets (database credentials)
infisical dynamic-secrets generate --provider postgres --secret-name db-creds
```

## Multi-domain configuration

Infisical supports three deployment options:

### US Cloud (default)
```bash
infisical config set API_URL https://api.infisical.com
# Default endpoint: app.infisical.com
```

### EU Cloud
```bash
infisical config set API_URL https://eu-api.infisical.com
# Default endpoint: eu.infisical.com
```

### Self-hosted
```bash
infisical config set API_URL https://your-infisical-instance.com/api
infisical config set INFISICAL_URL https://your-infisical-instance.com
```

## Authentication methods

### User authentication

**Browser-based login (interactive):**
```bash
infisical login                    # Opens browser for OAuth
infisical login --email user@example.com  # Email magic link
```

**Universal Auth (machine-to-machine):**
```bash
infisical login universal-auth --client-id <client-id> --client-secret <secret>
```

### Machine identity authentication

**Kubernetes:**
```bash
infisical login kubernetes --identity-id <identity-id> --service-account-name <sa-name> --namespace <ns>
```

**Azure:**
```bash
infisical login azure --identity-id <identity-id> --client-id <client-id> --tenant-id <tenant-id>
```

**GCP:**
```bash
infisical login gcp --identity-id <identity-id> --service-account-email <email>
```

**AWS:**
```bash
infisical login aws --identity-id <identity-id> --role-arn <arn>
```

**OIDC:**
```bash
infisical login oidc --identity-id <identity-id> --issuer-url <url> --role-arn <arn>
```

**JWT:**
```bash
infisical login jwt --token <jwt-token>
```

## Environment variables

| Variable | Description | Required |
|----------|-------------|----------|
| `INFISICAL_TOKEN` | Personal access token or service token | No* |
| `INFISICAL_API_URL` | API endpoint URL (default: https://api.infisical.com) | No |
| `INFISICAL_DISABLE_UPDATE_CHECK` | Disable version update check (set to 1) | No |
| `INFISICAL_CUSTOM_HEADERS` | Custom headers as JSON for API requests | No |
| `INFISICAL_LOG_LEVEL` | Debug logging level (debug, info, warn, error) | No |

*If not set, uses token from `infisical config` after login

## Skill organization

This skill routes to specialized sub-skills by Infisical domain:

**Authentication & Identity:**
- `infisical-auth` - Login, logout, authentication methods
- `infisical-service-token` - Service token creation and management
- `infisical-token` - Personal access tokens

**Secrets Management:**
- `infisical-secrets` - CRUD operations for secrets
- `infisical-dynamic-secrets` - Dynamic secrets generation (databases, cloud)
- `infisical-run` - Command execution with secrets injection

**Infrastructure:**
- `infisical-ssh` - SSH certificate management
- `infisical-pam` - Privileged access management
- `infisical-kmip` - KMIP server operations
- `infisical-vault` - Vault integration and management

**Project & Configuration:**
- `infisical-init` - Project initialization
- `infisical-bootstrap` - Project bootstrap and setup
- `infisical-export` - Export secrets and configuration

**Advanced Operations:**
- `infisical-gateway` - API gateway configuration
- `infisical-relay` - Relay server management
- `infisical-migration` - Data migration tools

## Command reference

| Command | Description |
|---------|-------------|
| `infisical login` | Authenticate with Infisical |
| `infisical init` | Initialize project with Infisical |
| `infisical run` | Run command with injected secrets |
| `infisical secrets` | Manage secrets (list, get, set, delete) |
| `infisical dynamic-secrets` | Generate dynamic database/cloud credentials |
| `infisical ssh` | Manage SSH certificates |
| `infisical pam` | Privileged access management operations |
| `infisical gateway` | Configure API gateway |
| `infisical kmip` | KMIP server operations |
| `infisical relay` | Relay server management |
| `infisical bootstrap` | Bootstrap project configuration |
| `infisical export` | Export secrets to file |
| `infisical token` | Generate personal access tokens |
| `infisical service-token` | Manage service tokens |
| `infisical vault` | Vault integration |
| `infisical user` | User management commands |
| `infisical reset` | Reset local configuration |

## Common workflows

### Daily development

```bash
# Set up a new project
infisical login
infisical init

# Add secrets
infisical secrets set DATABASE_URL=postgres://localhost/db --env=development
infisical secrets set API_KEY=sk-xxx --env=production

# Run application with secrets
infisical run --env=development -- nodemon src/index.js
```

### CI/CD integration

```bash
# Using service token
export INFISICAL_TOKEN=<service-token>
infisical run --env=production -- npm run build

# Using dynamic secrets for database
infisical dynamic-secrets generate --provider postgres --secret-name db-creds --env=production
```

### Managing multiple environments

```bash
# Development
infisical secrets --env=development --export > .env.development

# Staging
infisical secrets --env=staging --export > .env.staging

# Production
infisical secrets --env=production --export > .env.production
```

## Decision Trees

### "Which authentication method should I use?"

```
Use case:
├─ Interactive user workflow → Browser login (infisical login)
├─ CI/CD pipeline → Service token (infisical service-token)
├─ Kubernetes pod → Kubernetes auth (infisical login kubernetes)
├─ Cloud workload (Azure/GCP/AWS) → Cloud identity auth
└─ Machine-to-machine → Universal Auth or JWT
```

### "How to inject secrets into my application?"

```
Approach:
├─ Wrapper command → infisical run -- node app.js
├─ Export to .env → infisical secrets --env=prod --export > .env
├─ Direct lookup → infisical secrets get KEY_NAME
└─ Docker/containers → Use infisical-init in Dockerfile
```

### "Need dynamic database credentials?"

```
Database type:
├─ PostgreSQL → infisical dynamic-secrets generate --provider postgres
├─ MySQL → infisical dynamic-secrets generate --provider mysql
├─ MongoDB → infisical dynamic-secrets generate --provider mongodb
├─ Redis → infisical dynamic-secrets generate --provider redis
└─ AWS (RDS) → infisical dynamic-secrets generate --provider aws-rds
```

## Security considerations

- **Token storage**: Tokens are stored in `~/.config/infisical/config.json` - protect this file
- **Service tokens**: Use minimal required scopes for automation
- **Audit logs**: Review Infisical audit logs for secret access
- **Rotation**: Regularly rotate tokens and secrets
- **Environment separation**: Use different projects/secrets for dev/staging/prod

## Related Skills

**For secrets injection in other tools:**
- Use `infisical-run` wrapper for command execution
- Use `infisical-export` for .env file generation
- Script: `scripts/secrets-inject.sh` automates env file creation

**For CI/CD:**
- Use service tokens with minimal scope
- Use `infisical-secrets` for secret management
- Reference: Infisical CI/CD documentation for provider-specific guides

**For cloud infrastructure:**
- Use `infisical-dynamic-secrets` for credential rotation
- Use `infisical-vault` for HashiCorp Vault integration
- Use `infisical-kmip` for certificate management