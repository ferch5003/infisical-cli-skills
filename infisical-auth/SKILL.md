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
| `infisical login` | Authenticate with browser or credentials |
| `infisical logout` | Clear local authentication |
| `infisical user` | Display current user info |
| `infisical reset` | Reset local configuration |

## login

Authenticate with Infisical. Supports multiple authentication methods.

### Browser login (interactive)

```bash
# Open browser for OAuth (default)
infisical login

# Email magic link
infisical login --email user@example.com
```

**Flags:**
- `--email` - Email for magic link authentication
- `--interactive` - Force interactive mode
- `--domain` - Custom Infisical instance URL
- `--organization-slug` - Auto-select organization

### Universal Auth (machine-to-machine)

```bash
infisical login universal-auth \
  --client-id <client-id> \
  --client-secret <client-secret>
```

**Flags:**
- `--client-id` (required) - Universal auth client ID
- `--client-secret` (required) - Universal auth client secret

**Example with organization:**
```bash
infisical login universal-auth \
  --client-id ua_xxxxx \
  --client-secret xxxxx \
  --organization-slug my-org
```

### Kubernetes authentication

```bash
infisical login kubernetes \
  --identity-id <identity-id> \
  --service-account-name <sa-name> \
  --namespace <namespace>
```

**Flags:**
- `--identity-id` (required) - Machine identity ID
- `--service-account-name` (required) - Kubernetes service account name
- `--namespace` - Kubernetes namespace (default: default)

**Example:**
```bash
infisical login kubernetes \
  --identity-id ia_xxxxx \
  --service-account-name my-app \
  --namespace production
```

### Azure authentication

```bash
infisical login azure \
  --identity-id <identity-id> \
  --client-id <client-id> \
  --tenant-id <tenant-id>
```

**Flags:**
- `--identity-id` (required) - Machine identity ID
- `--client-id` (required) - Azure client ID
- `--tenant-id` (required) - Azure tenant ID

**Example:**
```bash
infisical login azure \
  --identity-id ia_xxxxx \
  --client-id xxxxx-xxxx-xxxx \
  --tenant-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### GCP authentication

```bash
infisical login gcp \
  --identity-id <identity-id> \
  --service-account-email <email>
```

**Flags:**
- `--identity-id` (required) - Machine identity ID
- `--service-account-email` (required) - GCP service account email

**Example:**
```bash
infisical login gcp \
  --identity-id ia_xxxxx \
  --service-account-email my-app@project.iam.gserviceaccount.com
```

### AWS authentication

```bash
infisical login aws \
  --identity-id <identity-id> \
  --role-arn <arn>
```

**Flags:**
- `--identity-id` (required) - Machine identity ID
- `--role-arn` (required) - AWS role ARN

**Example:**
```bash
infisical login aws \
  --identity-id ia_xxxxx \
  --role-arn arn:aws:iam::123456789012:role/MyInfisicalRole
```

### OIDC authentication

```bash
infisical login oidc \
  --identity-id <identity-id> \
  --issuer-url <url> \
  --role-arn <arn>
```

**Flags:**
- `--identity-id` (required) - Machine identity ID
- `--issuer-url` (required) - OIDC issuer URL
- `--role-arn` - AWS role ARN (optional)

**Example:**
```bash
infisical login oidc \
  --identity-id ia_xxxxx \
  --issuer-url https://accounts.google.com \
  --role-arn arn:aws:iam::123456789012:role/OIDCRole
```

### JWT authentication

```bash
infisical login jwt --token <jwt-token>
```

**Flags:**
- `--token` (required) - JWT token

**Example:**
```bash
infisical login jwt --token eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Common flags for login

| Flag | Description |
|------|-------------|
| `--method` | Auth method (browser, universal-auth, kubernetes, azure, gcp, aws, oidc, jwt) |
| `--client-id` | Client ID for universal-auth |
| `--client-secret` | Client secret for universal-auth |
| `--email` | Email for magic link |
| `--password` | Password (not recommended) |
| `--interactive` | Force interactive prompt |
| `--plain` | Output token only (for scripting) |
| `--silent` | Suppress non-essential output |
| `--domain` | Custom Infisical instance |
| `--organization-slug` | Auto-select organization |

### Token output for scripting

Use `--plain --silent` to output only the token for scripting:

```bash
# Get token for use in scripts
TOKEN=$(infisical login --plain --silent --method universal-auth --client-id xxx --client-secret xxx)
echo $TOKEN

# Use in CI/CD
export INFISICAL_TOKEN=$(infisical login --plain --silent --method universal-auth --client-id $CLIENT_ID --client-secret $CLIENT_SECRET)
```

## logout

Clear local authentication credentials.

```bash
# Logout from current session
infisical logout

# Logout and clear cached data
infisical logout --clear-cache
```

**Flags:**
- `--clear-cache` - Also clear cached secrets and data
- `--global` - Clear global configuration (all domains)

## user

Display current authenticated user information.

```bash
# Show current user
infisical user

# Get user email only
infisical user --email

# Get user ID only
infisical user --id
```

**Flags:**
- `--email` - Output only email
- `--id` - Output only user ID
- `--name` - Output only name
- `--organization` - Show organization membership

## reset

Reset local Infisical configuration.

```bash
# Reset to defaults
infisical reset

# Reset and re-authenticate
infisical reset --reauth
```

**Flags:**
- `--reauth` - Automatically start re-authentication after reset
- `--force` - Skip confirmation prompt

## Authentication workflow decisions

```
Need authentication for:
├─ Local development → infisical login (browser)
├─ CI/CD pipeline → Service token or Universal Auth
├─ Kubernetes pod → Kubernetes auth
├─ Azure VM/workload → Azure auth
├─ GCP VM/workload → GCP auth
├─ AWS EC2/Lambda → AWS auth
├─ OIDC provider → OIDC auth
└─ Existing JWT → JWT auth
```

## Environment variables

| Variable | Description |
|----------|-------------|
| `INFISICAL_TOKEN` | Use specific token (bypasses login) |
| `INFISICAL_API_URL` | API endpoint (default: https://api.infisical.com) |
| `INFISICAL_DISABLE_UPDATE_CHECK` | Set to 1 to disable update check |

## Token storage

After successful login, credentials are stored in:
- **Linux/macOS**: `~/.config/infisical/config.json`
- **Windows**: `%APPDATA%/infisical/config.json`

**Security:** Protect this file - it contains your authentication tokens.

## Related skills

- `infisical-service-token` - Create service tokens for CI/CD
- `infisical-token` - Personal access tokens
- `infisical-secrets` - Manage secrets after authentication