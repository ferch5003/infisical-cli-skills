---
name: infisical-export
description: Export secrets to various formats. Use when user mentions export secrets, env file, .env export, secrets to json/yaml.
metadata:
  openclaw:
    requires:
      bins: [infisical]
---

# Infisical Export

Export secrets to files in multiple formats.

## Quick reference

| Command | Description |
|---------|-------------|
| `infisical export` | Export all secrets |
| `infisical secrets --export` | Export environment secrets |

## export command

```bash
# Export all secrets to .env format
infisical export

# Export to JSON
infisical export --format json

# Export to YAML
infisical export --format yaml

# Export specific environment
infisical export --env=production

# Export to file
infisical export --path ./secrets.env

# Include references
infisical export --include-references
```

**Flags:**
- `--env` - Environment name (development, staging, production)
- `--format` - Output format (env, json, yaml, dotenv)
- `--path` - Output file path
- `--include-references` - Include secret references
- `--uppercase` - Uppercase keys
- `--escape` - Escape special characters

## Format examples

### env/dotenv (default)

```
DATABASE_URL=postgres://localhost:5432/db
API_KEY=sk-xxxxx
SECRET_TOKEN=yyyy
```

### JSON

```json
{
  "DATABASE_URL": "postgres://localhost:5432/db",
  "API_KEY": "sk-xxxxx"
}
```

### YAML

```yaml
DATABASE_URL: postgres://localhost:5432/db
API_KEY: sk-xxxxx
```

## Environment-specific export

```bash
# Development
infisical export --env=development --format env > .env.development

# Staging
infisical export --env=staging --format env > .env.staging

# Production
infisical export --env=production --format env > .env.production
```

## CI/CD integration

```bash
# Export and source in bash
source <(infisical export --env=production --format env)

# Export for Docker
infisical export --env=production --format env > .env
```

## Combined with secrets get

```bash
# Export single secret
infisical secrets get API_KEY --env=production
```