---
name: infisical-export
description: Export secrets to various formats. Use when user mentions export secrets, env file, .env export, secrets to json/csv.
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
        scope: fetch and export secrets
---

# infisical-export

Export secrets from Infisical to files in multiple formats.

## Command

```bash
infisical export [flags]
```

## Quick Examples

```bash
# Export to .env (dotenv format — default)
infisical export --env=production > .env

# Export to JSON
infisical export --env=production --format=json > secrets.json

# Export to CSV
infisical export --env=production --format=csv > secrets.csv

# Write directly to file
infisical export --env=production --output-file=./secrets.env
infisical export --env=production --format=json --output-file=./secrets.json

# Export from a folder path
infisical export --path=/backend --env=production
```

## Flags

| Flag | Description | Default |
|------|-------------|---------|
| `-e, --env <env>` | Environment name | `dev` |
| `--path <path>` | Folder path within project | `/` |
| `-f, --format <format>` | Output format: `dotenv`, `json`, `csv` | `dotenv` |
| `-o, --output-file <path>` | Write output to file (instead of stdout) | stdout |
| `--expand` | Expand secret references (`${SECRET_KEY}`) | `true` |
| `--include-imports` | Include imported linked secrets | `true` |
| `--secret-overriding` | Prefer personal secrets over shared | `true` |
| `--tags <slug,slug>` | Filter secrets by tag slugs | — |
| `--token <token>` | Service/machine identity token | auto |
| `--projectId <id>` | Project ID (machine identity auth) | auto |
| `--template <file>` | Template file to render secrets | — |

## Formats

### dotenv (default)

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

### CSV

```csv
key,value
DATABASE_URL,postgres://localhost:5432/db
API_KEY,sk-xxxxx
```

## Environment-Specific Export

```bash
# Development
infisical export --env=development --format=dotenv > .env.development

# Staging
infisical export --env=staging --format=dotenv > .env.staging

# Production
infisical export --env=production --format=dotenv > .env.production

# All formats
infisical export --env=production --format=dotenv --output-file=.env
infisical export --env=production --format=json --output-file=secrets.json
infisical export --env=production --format=csv --output-file=secrets.csv
```

## Template Rendering

Use `--template` to render secrets through a custom template file:

```bash
# template.txt
# Database configuration
DB_HOST={{.DATABASE_HOST}}
DB_PORT={{.DATABASE_PORT}}
DB_USER={{.DATABASE_USER}}
DB_PASS={{.DATABASE_PASSWORD}}

infisical export --env=production --template=./template.txt --output-file=./config.txt
```

## CI/CD Integration

```bash
# Export and source in bash
source <(infisical export --env=production --format=dotenv)

# Docker — write .env file
infisical export --env=production --format=dotenv --output-file=.env

# CI with token
export INFISICAL_TOKEN=<service-token>
infisical export --env=production --format=dotenv > .env
```

## Filtering

```bash
# Only secrets tagged "api"
infisical export --tags=api --env=production

# From specific folder
infisical export --path=/backend/services --env=production
```

## Common Patterns

```bash
# Development workflow
infisical export --env=development --format=dotenv > .env
source .env
npm run dev

# Docker Compose
infisical export --env=production --format=dotenv --output-file=.env
docker-compose --env-file .env up

# Backup all envs
for env in development staging production; do
  infisical export --env=$env --format=json --output-file=secrets-$env.json
done
```

## Notes

- **No `yaml` format** — only `dotenv`, `json`, and `csv` are supported
- **`--format=dotenv`** is the same as `.env` format
- **`--output-file`** writes to disk, omitting it prints to stdout
- **`--path`** is the project folder path, **not** the output file path