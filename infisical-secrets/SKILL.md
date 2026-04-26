---
name: infisical-secrets
description: Infisical CLI secrets CRUD operations. Use when user mentions infisical secrets, manage secrets, secret management, list secrets, get secret, set secret, delete secret.
metadata:
  openclaw:
    requires:
      bins: [infisical]
      credentials:
        - name: INFISICAL_TOKEN
          description: Infisical personal access token or service token
          required: true
    network:
      - description: Outbound HTTPS to Infisical API
        scope: secrets CRUD operations
---

# infisical-secrets

CRUD operations for secrets in Infisical.

## Triggers

- `infisical secrets` / `infisical secrets list`
- `infisical secrets get`
- `infisical secrets set`
- `infisical secrets delete`
- `infisical secrets folders`

## Global Flags

These flags apply to all `infisical secrets` subcommands.

| Flag | Description | Default |
|------|-------------|---------|
| `--env <env>` | Environment name | `dev` |
| `--path <path>` | Folder path within project | `/` |
| `--projectId <id>` | Project ID (for machine identity auth) | auto |
| `--token <token>` | Service/machine identity token | auto |
| `--expand` | Expand secret references (`${SECRET_KEY}`) | `true` |
| `--include-imports` | Include imported linked secrets | `true` |
| `--recursive` | Fetch secrets from all sub-folders | `false` |
| `--secret-overriding` | Prefer personal secrets over shared | `true` |
| `--tags <slug,slug>` | Filter by tag slugs | — |
| `--output <format>` | Output format: `yaml`, `json`, `dotenv` | — |
| `--plain` | Plain output (one per line) — **deprecated**, use `--output dotenv` | — |
| `--silent` | Suppress non-essential output | — |

## List secrets

**Note:** There is no `list` subcommand. `infisical secrets` (no subcommand) defaults to listing.

```bash
# List all secrets in default env (dev) at root path
infisical secrets

# List secrets in specific environment
infisical secrets --env=production

# List secrets in specific project
infisical secrets --projectId=<project-id> --env=staging

# List secrets in a folder path
infisical secrets --path=/backend --env=development

# List with expanded references
infisical secrets --expand --env=dev

# List from all sub-folders
infisical secrets --recursive --env=production

# Filter by tags
infisical secrets --tags=api,frontend --env=production

# JSON output
infisical secrets --output=json --env=production

# Plain dotenv output (deprecated --plain)
infisical secrets --output=dotenv --env=production
```

## Get secret

Takes **positional arguments** — secret names passed directly, not via `--secret-name`.

```bash
# Get one secret
infisical secrets get DATABASE_URL

# Get multiple secrets at once
infisical secrets get DATABASE_URL API_KEY REDIS_URL

# Get in production environment
infisical secrets get API_KEY --env=production

# Get in a folder path
infisical secrets get API_KEY --path=/backend --env=production

# Get with project specified
infisical secrets get DB_PASSWORD --projectId=<id> --env=dev

# JSON output
infisical secrets get API_KEY --output=json --env=production

# Include imports
infisical secrets get API_KEY --include-imports
```

## Set secret

Takes **`KEY=VALUE` pairs as positional arguments**. There are no `--secret-name` or `--secret-value` flags.

```bash
# Set a single secret
infisical secrets set API_KEY=abc123

# Set multiple secrets at once
infisical secrets set API_KEY=abc123 DATABASE_URL=postgres://localhost/db

# Set in specific environment
infisical secrets set DATABASE_URL=postgres://... --env=production

# Set in a folder path
infisical secrets set STRIPE_KEY=sk_xxx --path=/payments --env=production

# Set with project ID
infisical secrets set DB_PASSWORD=secret --projectId=<id> --env=staging

# Set secret type (personal or shared — defaults to shared)
infisical secrets set API_KEY=value --type=shared --env=production
infisical secrets set MY_SECRET=value --type=personal --env=production

# Load secrets from a .env or YAML file (mutually exclusive with inline pairs)
infisical secrets set --file=./secrets.env --env=production
infisical secrets set --file=./secrets.yaml --env=production
```

**Key flags for set:**
- `--type personal|shared` — defaults to `shared`
- `--file <path>` — load from `.env` or YAML file
- `--path` — folder path (default `/`)

## Delete secret

Takes **positional arguments** — secret names passed directly, not via `--secret-name`.

```bash
# Delete a secret (deletes personal secret by default)
infisical secrets delete API_KEY

# Delete multiple secrets at once
infisical secrets delete API_KEY DB_PASSWORD OLD_KEY

# Delete shared secret
infisical secrets delete API_KEY --type=shared --env=production

# Delete personal secret
infisical secrets delete API_KEY --type=personal --env=production

# Delete in specific environment
infisical secrets delete DB_PASSWORD --env=production

# Delete from a folder path
infisical secrets delete OLD_SECRET --path=/legacy --env=staging
```

**Important:** The **default for `delete` is `--type=personal`**, while `set` defaults to `shared`. This is a common source of confusion.

## Folders

### List folders

Use `infisical secrets folders get` (not bare `infisical secrets folders`).

```bash
# List folders at root
infisical secrets folders get

# List folders in specific environment
infisical secrets folders get --env=production

# List folders at a specific path
infisical secrets folders get --path=/backend --env=development

# List folders in specific project
infisical secrets folders get --projectId=<id> --env=staging
```

### Create folder

Use `--name` (or `-n`) for the folder name and `--path` for the **parent path**.

```bash
# Create folder at root
infisical secrets folders create --name=backend

# Create folder at root (shorthand)
infisical secrets folders create -n backend --env=production

# Create folder inside a path
infisical secrets folders create --name=services --path=/backend --env=production

# Create nested structure
infisical secrets folders create --name=api --path=/backend --env=production
```

### Delete folder

Use `--name` (or `-n`) for the folder name and `--path` for the **parent path**.

```bash
# Delete folder at root
infisical secrets folders delete --name=backend

# Delete folder inside a path
infisical secrets folders delete --name=services --path=/backend --env=production
```

## Generate .env file

```bash
# Generate .env from secrets
infisical secrets generate-example-env

# Generate for specific environment
infisical secrets generate-example-env --env=production

# Generate from a folder
infisical secrets generate-example-env --path=/backend --env=production

# Redirect to file
infisical secrets generate-example-env --env=production > .env.production
```

## Output Formats

| Format | Flag | Use case |
|--------|------|----------|
| Plain (one per line) | `--output dotenv` | `.env` files, scripting |
| JSON | `--output json` | API integration, parsing |
| YAML | `--output yaml` | Config files, CI/CD |
| Table (default) | — | Human readability |

## Common Patterns

### Export secrets to .env
```bash
infisical secrets --env=production --output=dotenv > .env
```

### Batch update from file
```bash
# Create a secrets.env file, then:
infisical secrets set --file=./secrets.env --env=production
```

### Script with secrets
```bash
#!/bin/bash
infisical secrets --env=production --output=dotenv > .env
source .env
./start-server.sh
```

## Important Notes

- **`--secret-name` and `--secret-value` flags do not exist** — use positional arguments
- **`--plain` is deprecated** — use `--output dotenv` instead
- **`--expand` defaults to `true`** — secrets referencing other secrets (`${OTHER_SECRET}`) are auto-resolved
- **`--include-imports` defaults to `true`** — linked/imported secrets are included
- **Delete defaults to `type=personal`** — use `--type=shared` to delete shared secrets
- **Set defaults to `type=shared`** — use `--type=personal` for personal overrides
- **`--path` in folder commands is the parent path**, `--name` is the folder name itself