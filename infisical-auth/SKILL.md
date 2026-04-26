---
name: infisical-auth
description: Infisical CLI authentication commands (login, logout, reset, user). Use when user mentions infisical login, infisical logout, infisical user, authentication, or auth methods.
metadata:
  openclaw:
    requires:
      bins: [infisical]
      credentials:
        - name: INFISICAL_TOKEN
          description: Infisical personal access token or service token
          required: false
          fallback: infisical config (set via infisical login)
    network:
      - description: Outbound HTTPS to Infisical API (app.infisical.com by default)
        scope: authenticated API calls
---

# Infisical Auth

Authentication commands for Infisical CLI.

## Quick reference

| Command | Description |
|---------|-------------|
| `infisical login` | Authenticate with Infisical (multiple methods) |
| `infisical reset` | Clear local credentials and configuration |
| `infisical user get` | Display current authenticated user info |
| `infisical user switch` | Switch between Infisical profiles |
| `infisical user get token` | Get the current access token |

## login

Authenticate with Infisical. Supports multiple authentication methods.

### Browser login (interactive)

```bash
# Open browser for OAuth (default)
infisical login

# Email magic link
infisical login --email user@example.com

# Organization auto-select
infisical login --organization-slug my-org
```

**Flags:**
- `--email` - Email for magic link authentication
- `--organization-id` - Organization ID for 'user' login method
- `--organization-slug` - Auto-select organization (sub-org scope for machine identity)
- `--interactive` - Force interactive mode
- `--domain` - Custom Infisical instance URL
- `--password` - Password (not recommended)

### Universal Auth (machine-to-machine)

```bash
infisical login --method universal-auth \
  --client-id <client-id> \
  --client-secret <client-secret>
```

**Flags:**
- `--method universal-auth` (required) - Must specify the method
- `--client-id` (required) - Universal auth client ID
- `--client-secret` (required) - Universal auth client secret
- `--organization-slug` - Scope session to a sub-organization

**Example with organization:**
```bash
infisical login \
  --method universal-auth \
  --client-id ua_xxxxx \
  --client-secret xxxxx \
  --organization-slug my-org
```

### Kubernetes authentication

```bash
infisical login --method kubernetes \
  --machine-identity-id <identity-id> \
  --service-account-token-path <token-path>
```

**Flags:**
- `--method kubernetes` (required)
- `--machine-identity-id` (required) - Machine identity ID (`ia_xxxxx`)
- `--service-account-token-path` (required) - Path to the service account token file
- `--organization-slug` - Scope to sub-organization

**Example:**
```bash
infisical login \
  --method kubernetes \
  --machine-identity-id ia_xxxxx \
  --service-account-token-path /var/run/secrets/tokens/service-account-token
```

### Azure authentication

```bash
infisical login --method azure \
  --machine-identity-id <identity-id> \
  --client-id <azure-client-id> \
  --tenant-id <azure-tenant-id>
```

**Flags:**
- `--method azure` (required)
- `--machine-identity-id` (required) - Machine identity ID (`ia_xxxxx`)
- `--client-id` (required) - Azure client ID (application ID)
- `--tenant-id` (required) - Azure tenant ID

### GCP authentication (two methods)

**Method 1: GCP ID Token (via OIDC)**
```bash
infisical login --method gcp-id-token --jwt <id-token>
```

**Method 2: GCP IAM (via service account key)**
```bash
infisical login --method gcp-iam \
  --machine-identity-id <identity-id> \
  --service-account-key-file-path <key-file-path>
```

**Flags for gcp-iam:**
- `--method gcp-iam` (required)
- `--machine-identity-id` (required) - Machine identity ID
- `--service-account-key-file-path` (required) - Path to the GCP service account key JSON file

### AWS authentication

```bash
infisical login --method aws-iam \
  --machine-identity-id <identity-id> \
  --role-arn <arn>
```

**Flags:**
- `--method aws-iam` (required)
- `--machine-identity-id` (required) - Machine identity ID (`ia_xxxxx`)
- `--role-arn` (required) - AWS role ARN

**Example:**
```bash
infisical login \
  --method aws-iam \
  --machine-identity-id ia_xxxxx \
  --role-arn arn:aws:iam::123456789012:role/MyInfisicalRole
```

### OIDC authentication

```bash
infisical login --method oidc-auth --jwt <oidc-token>
```

**Flags:**
- `--method oidc-auth` (required)
- `--jwt` (required) - OIDC ID token

**Example:**
```bash
infisical login \
  --method oidc-auth \
  --jwt eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

### JWT / OIDC token authentication

```bash
infisical login --method jwt-auth --jwt <token>
```

**Flags:**
- `--method jwt-auth` (required)
- `--jwt` (required) - JWT token

## logout

There is **no `infisical logout` command**. Use `infisical reset` instead to clear local credentials.

## reset

Clear local Infisical configuration and credentials.

```bash
# Clear all Infisical data from this machine
infisical reset
```

## user get

Display current authenticated user information.

```bash
# Show current user details
infisical user get

# Get current access token (useful for scripting)
infisical user get token --plain
```

**Subcommands:**
- `infisical user get` - Get user profile info (email, name, etc.)
- `infisical user get token` - Get the access token (use `--plain` for scripting)

## user switch

Switch between multiple Infisical profiles.

```bash
infisical user switch
```

## Authentication workflow decisions

```
Need authentication for:
├─ Local development → infisical login (browser, user method)
├─ CI/CD pipeline → --method universal-auth
├─ Kubernetes pod → --method kubernetes --machine-identity-id
├─ Azure VM/workload → --method azure
├─ GCP VM (OIDC) → --method gcp-id-token --jwt
├─ GCP VM (IAM) → --method gcp-iam --machine-identity-id
├─ AWS EC2/Lambda → --method aws-iam --machine-identity-id
├─ OIDC provider → --method oidc-auth --jwt
└─ Existing JWT → --method jwt-auth --jwt
```

## Environment variables

| Variable | Description |
|----------|-------------|
| `INFISICAL_TOKEN` | Use specific token directly (bypasses login) |
| `INFISICAL_API_URL` | API endpoint (default: https://app.infisical.com/api) |
| `INFISICAL_DISABLE_UPDATE_CHECK` | Set to 1 to disable update check |

## Token output for scripting

Use `--plain --silent` with universal auth to output only the token:

```bash
# Get token for use in scripts
TOKEN=$(infisical login --plain --silent \
  --method universal-auth \
  --client-id xxx \
  --client-secret xxx)
echo $TOKEN

# Use in CI/CD
export INFISICAL_TOKEN=$(infisical login --plain --silent \
  --method universal-auth \
  --client-id $CLIENT_ID \
  --client-secret $CLIENT_SECRET)
```

## Token storage

After successful login, credentials are stored at:
- `infisical vault` - Manage token storage location
- Default Linux/macOS: `~/.config/infisical/config.json`
- Default Windows: `%APPDATA%/infisical/config.json`

Use `infisical vault` to configure custom storage paths.

**Security:** Protect this file — it contains your authentication tokens.

## Related skills

- `infisical-service-token` - Create service tokens for CI/CD
- `infisical-token` - Personal access tokens
- `infisical-secrets` - Manage secrets after authentication