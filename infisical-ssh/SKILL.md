---
name: infisical-ssh
description: SSH certificate management. Use when user mentions ssh certificates, ssh certs, SSH key signing.
metadata:
  openclaw:
    requires:
      bins: [infisical]
---

# Infisical SSH

Manage SSH certificates for server access.

## Quick reference

| Command | Description |
|---------|-------------|
| `infisical ssh certify` | Generate SSH certificate |
| `infisical ssh list` | List active certificates |
| `infisical ssh remove` | Revoke certificate |

## certify

Generate SSH certificate for server access.

```bash
# Basic certificate
infisical ssh certify --user ubuntu --hostname server1.example.com

# With TTL
infisical ssh certify \
  --user ubuntu \
  --hostname server1.example.com \
  --ttl 24h

# Multiple principals
infisical ssh certify \
  --user deploy \
  --principals ubuntu,root \
  --hostname server1.example.com

# From specific project
infisical ssh certify \
  --user app \
  --hostname prod-server.example.com \
  --env=production
```

**Flags:**
- `--user` (required) - SSH username
- `--hostname` (required) - Target hostname
- `--ttl` - Certificate validity (default: 24h)
- `--principals` - Additional principals
- `--env` - Environment
- `--output` - Output key path

## list

List active SSH certificates.

```bash
infisical ssh list
infisical ssh list --user ubuntu
infisical ssh list --hostname server1.example.com
```

**Flags:**
- `--user` - Filter by user
- `--hostname` - Filter by hostname
- `--expired` - Include expired certificates

## remove

Revoke SSH certificate.

```bash
infisical ssh remove --user ubuntu --hostname server1.example.com
infisical ssh remove --all
```

**Flags:**
- `--user` - Username
- `--hostname` - Hostname
- `--all` - Remove all certificates
- `--force` - Skip confirmation

## TTL format

```
1h    - 1 hour
8h    - 8 hours
24h   - 24 hours (default)
7d    - 7 days
30d   - 30 days
```

## SSH config example

```bash
# Add to ~/.ssh/config
Host server1
  HostName server1.example.com
  User ubuntu
  IdentityAgent ~/Library/Group\ Containers/com.infisical.ssh-agent.sock
```

## Use cases

```bash
# Generate short-lived cert for deployment
infisical ssh certify --user deploy --hostname prod.example.com --ttl 1h

# Generate long-term cert
infisical ssh certify --user admin --hostname admin.example.com --ttl 30d
```