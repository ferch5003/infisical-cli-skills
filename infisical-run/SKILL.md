# infisical-run

Run commands with Infisical secrets injected as environment variables.

## When to Use

Use when running applications that require secrets from Infisical without manually managing environment variables. Perfect for:
- Running scripts with API keys
- Starting servers with credentials
- Docker/podman commands needing secrets
- npm/node scripts, maven, python, docker-compose

## Triggers

- `infisical run`
- `inject secrets`
- `secrets injection`
- `run with secrets`

## Command

```bash
infisical run [flags] -- <your-command>
```

## Flags

| Flag | Type | Description |
|------|------|-------------|
| `--env` | string | Environment name (e.g., dev, staging, prod) |
| `--path` | string | Path to secrets (e.g., /my-app/config) |
| `--secret-override` | string | Override secrets via CLI (format: KEY=value) |
| `--silent` | bool | Suppress non-error output |

## Usage

```bash
# Run with environment
infisical run --env=dev -- node app.js

# Run with path
infisical run --path=/my-app/production -- npm start

# Run with secret override
infisical run --env=prod --secret-override=API_KEY=custom-key -- your-app-command

# Run silently
infisical run --silent -- python script.py
```

## How Secrets Are Injected

1. Infisical fetches secrets for the specified environment or path
2. Secrets are converted to environment variables
3. Your command runs with those env vars set
4. Child process inherits all Infisical secrets as `$SECRET_NAME`

### Example Injection

```
Infisical secret:  my-api-key: sk_live_abc123
Injected as:       MY_API_KEY=sk_live_abc123
```

### Common Use Cases

```bash
# Node.js with secrets
infisical run --env=dev -- node server.js

# Docker with secrets
infisical run --env=prod -- docker-compose up

# Python with secrets
infisical run --path=/backend -- python main.py

# npm scripts
infisical run --env=staging -- npm run dev
```

## Notes

- Child processes inherit all injected environment variables
- Secrets are never printed to stdout (--silent suppresses CLI output only)
- Override values take precedence over fetched secrets
- Run from project directory for local .infisical.json detection