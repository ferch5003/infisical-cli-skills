---
name: infisical-dynamic-secrets
description: Dynamic secrets generation for databases and cloud providers. Use when user mentions dynamic-secrets, dynamic secrets, temporary credentials, database credentials, rotating secrets.
metadata:
  openclaw:
    requires:
      bins: [infisical]
---

# Infisical Dynamic Secrets

Generate temporary credentials for databases and cloud services.

## Quick reference

| Command | Description |
|---------|-------------|
| `infisical dynamic-secrets list` | List dynamic secrets |
| `infisical dynamic-secrets create` | Generate new credentials |
| `infisical dynamic-secrets revoke` | Revoke credentials |

## list

List configured dynamic secrets providers.

```bash
infisical dynamic-secrets list
infisical dynamic-secrets list --env=production
```

**Flags:**
- `--env` - Environment name
- `--provider` - Filter by provider type

## create

Generate temporary credentials for a provider.

```bash
# PostgreSQL
infisical dynamic-secrets create \
  --provider postgres \
  --secret-name db-creds \
  --env=production

# MySQL
infisical dynamic-secrets create \
  --provider mysql \
  --secret-name mysql-creds \
  --env=production

# MongoDB
infisical dynamic-secrets create \
  --provider mongodb \
  --secret-name mongo-creds

# Redis
infisical dynamic-secrets create \
  --provider redis \
  --secret-name redis-creds
```

**Flags:**
- `--provider` (required) - Provider type
- `--secret-name` (required) - Secret name in Infisical
- `--env` - Environment
- `--ttl` - Time to live (default: 1h)
- `--username` - Override username
- `--database` - Database name

### Cloud providers

```bash
# AWS RDS
infisical dynamic-secrets create \
  --provider aws-rds \
  --secret-name rds-creds

# Azure SQL
infisical dynamic-secrets create \
  --provider azure-sql \
  --secret-name azure-db-creds

# GCP Cloud SQL
infisical dynamic-secrets create \
  --provider gcp-sql \
  --secret-name gcp-db-creds
```

### JDBC connection strings

```bash
# PostgreSQL JDBC
infisical dynamic-secrets create \
  --provider jdbc-postgres \
  --secret-name jdbc-conn

# MySQL JDBC
infisical dynamic-secrets create \
  --provider jdbc-mysql \
  --secret-name jdbc-mysql-conn
```

## revoke

Revoke active dynamic credentials.

```bash
infisical dynamic-secrets revoke --secret-name db-creds
infisical dynamic-secrets revoke --secret-name db-creds --env=production
```

**Flags:**
- `--secret-name` (required)
- `--env` - Environment
- `--force` - Skip confirmation

## Provider types

| Provider | Description |
|----------|-------------|
| `postgres` | PostgreSQL |
| `mysql` | MySQL/MariaDB |
| `mongodb` | MongoDB |
| `redis` | Redis |
| `aws-rds` | AWS RDS (PostgreSQL/MySQL) |
| `azure-sql` | Azure SQL Database |
| `gcp-sql` | GCP Cloud SQL |
| `jdbc-postgres` | JDBC PostgreSQL connection string |
| `jdbc-mysql` | JDBC MySQL connection string |

## TTL format

```
1h    - 1 hour
30m   - 30 minutes
24h   - 24 hours
7d    - 7 days
```

## Use cases

```bash
# Database connection in application
infisical run --env=production -- node db-connect.js

# Export credentials to .env
infisical dynamic-secrets create --provider postgres --secret-name db-creds
```