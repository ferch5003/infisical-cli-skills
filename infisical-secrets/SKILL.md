# Infisical Secrets Skill

## Metadata

- **name**: infisical-secrets
- **Description**: Infisical CLI secrets CRUD operations
- **Triggers**: infisical secrets, manage secrets, secret management

## Commands

### Primary Operations

| Operation | Command | Description |
|-----------|---------|-------------|
| List secrets | `infisical secrets list` | List all secrets in a project/folder |
| Get secret | `infisical secrets get` | Retrieve a specific secret value |
| Set secret | `infisical secrets set` | Create or update a secret |
| Delete secret | `infisical secrets delete` | Remove a secret |
| List folders | `infisical secrets folders` | List secret folders/path structure |
| Generate ENV | `infisical secrets generate-example-env` | Generate .env from secrets |

## Flags

| Flag | Description |
|------|-------------|
| `--projectId <id>` | Project ID to operate on |
| `--env <env>` | Environment (dev, staging, prod) |
| `--path <path>` | Folder path within project |
| `--expand` | Expand secret references |
| `--plain` | Output plain values (no masking) |
| `--silent` | Suppress non-essential output |
| `--secret-name <name>` | Name of secret to operate on |
| `--secret-value <value>` | Value for secret (set operation) |

## Sub-commands

### `infisical secrets list`

List all secrets in a project or path.

```bash
# List all secrets in default environment
infisical secrets list

# List secrets in specific environment
infisical secrets list --env=production

# List secrets in specific project
infisical secrets list --projectId=<project-id> --env=staging

# List secrets in a specific folder path
infisical secrets list --path=/backend --env=development

# List with expanded references
infisical secrets list --expand --env=dev
```

### `infisical secrets get`

Retrieve a single secret value.

```bash
# Get secret by name
infisical secrets get DATABASE_URL

# Get secret in production
infisical secrets get API_KEY --env=production

# Get with project specified
infisical secrets get --secret-name=DB_PASSWORD --projectId=<project-id> --env=dev

# Get plain output (no masking)
infisical secrets get SECRET_VALUE --plain
```

### `infisical secrets set`

Create or update a secret.

```bash
# Set a secret (interactive or inline)
infisical secrets set API_KEY=abc123

# Set in specific environment
infisical secrets set DATABASE_URL=postgres://... --env=production

# Set with explicit project and path
infisical secrets set --secret-name=STRIPE_KEY --secret-value=sk_xxx --projectId=<id> --env=staging --path=/payments

# Set multiple secrets
infisical secrets set HOST=localhost PORT=3000
```

### `infisical secrets delete`

Remove a secret.

```bash
# Delete a secret
infisical secrets delete API_KEY

# Delete in specific environment
infisical secrets delete DB_PASSWORD --env=production

# Delete with full path
infisical secrets delete --secret-name=OLD_SECRET --projectId=<id> --env=staging --path=/legacy
```

### `infisical secrets folders`

Manage folder structure for secrets organization.

```bash
# List folders
infisical secrets folders

# List folders in specific project
infisical secrets folders --projectId=<project-id> --env=development

# List folders at path
infisical secrets folders --path=/backend/services
```

### `infisical secrets folders create`

Create a new folder for organizing secrets.

```bash
# Create a new folder
infisical secrets folders create --path=/backend

# Create nested folder
infisical secrets folders create --path=/backend/services/api

# Create folder in specific environment
infisical secrets folders create --path=/legacy --env=production
```

### `infisical secrets folders delete`

Delete a folder and optionally its secrets.

```bash
# Delete an empty folder
infisical secrets folders delete --path=/backend/temp

# Delete folder with confirmation
infisical secrets folders delete --path=/legacy --force

# Delete folder in specific environment
infisical secrets folders delete --path=/old --env=staging

# Delete folder and all secrets within
infisical secrets folders delete --path=/legacy --recursive
```

### `infisical secrets generate-example-env`

Generate a `.env` file from existing secrets.

```bash
# Generate .env from secrets
infisical secrets generate-example-env

# Generate for specific environment
infisical secrets generate-example-env --env=production

# Generate with specific project
infisical secrets generate-example-env --projectId=<id> --env=staging

# Generate and save to file
infisical secrets generate-example-env > .env.staging
```

## Common Patterns

### Batch Operations

```bash
# Export all secrets to .env
infisical secrets list --plain --env=production > .env

# Sync secrets between environments
SECRETS=$(infisical secrets list --env=staging --plain)
echo "$SECRETS" | while read line; do
  key=$(echo $line | cut -d= -f1)
  value=$(echo $line | cut -d= -f2)
  infisical secrets set "$key=$value" --env=production
done
```

### Script Integration

```bash
#!/bin/bash
# Fetch secrets and run app
infisical secrets list --env=production --plain --path=/api > .env
source .env
./start-server.sh
```

## Tips

- Use `--plain` flag when output is piped or redirected
- Use `--silent` for cleaner CI/CD output
- Organize secrets in folders matching your project structure
- Use `--expand` to resolve secret references in values
- Default environment is usually `dev` or controlled by config