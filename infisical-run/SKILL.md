---
name: infisical-run
description: Run commands with Infisical secrets injected as environment variables. Use when user mentions infisical run, inject secrets, secrets injection, run with secrets, run with env vars.
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
        scope: fetch secrets before running command
---

# infisical-run

Inject Infisical secrets as environment variables into any command.

## When to Use

- Running applications that need secrets without manual env var management
- CI/CD pipelines that need secrets injected before running scripts
- Docker/podman commands needing secrets
- Development with `nodemon`, `python`, `npm`, etc.

## Command

```bash
infisical run [flags] -- <your-command>
```

## Quick Examples

```bash
# Run Node.js with secrets
infisical run --env=dev -- node server.js

# Run with a folder path
infisical run --path=/backend -- npm start

# Run chained commands (no -- needed)
infisical run -c "npm install && npm run dev"

# Run with watch (auto-reload on secret change)
infisical run --watch -- npm run dev

# Silent output
infisical run --silent -- python script.py
```

## Flags

| Flag | Description | Default |
|------|-------------|---------|
| `-e, --env <env>` | Environment name | `dev` |
| `--path <path>` | Folder path within project | `/` |
| `--expand` | Expand secret references (`${SECRET_KEY}`) | `true` |
| `--include-imports` | Include imported linked secrets | `true` |
| `--recursive` | Fetch secrets from all sub-folders | `false` |
| `--secret-overriding` | Prefer personal secrets over shared | `true` |
| `--tags <slug,slug>` | Filter secrets by tag slugs | — |
| `--token <token>` | Service/machine identity token | auto |
| `--projectId <id>` | Project ID (machine identity auth) | auto |
| `--project-config-dir <dir>` | Directory containing `.infisical.json` | current dir |
| `-c, --command <cmd>` | Chained shell commands to execute | — |
| `--watch` | Enable auto-reload when secrets change | `false` |
| `--watch-interval <secs>` | Seconds between secret checks (watch mode) | `10` |
| `--silent` | Suppress non-error output | `false` |

## Common Use Cases

### Node.js / JavaScript

```bash
infisical run --env=production -- node server.js
infisical run --env=dev -- nodemon src/index.js
infisical run --env=staging -- npm run build
```

### Python

```bash
infisical run --env=production -- python manage.py runserver
infisical run --path=/backend -- python main.py
```

### Docker / Docker Compose

```bash
infisical run --env=production -- docker-compose up
infisical run --env=prod -- docker build -t myapp .
infisical run --env=dev -- docker run myapp
```

### Chained commands (no -- needed)

```bash
infisical run -c "npm install && npm run build && echo done"
infisical run -c "pip install -r requirements.txt && python app.py"
```

### Development with watch

```bash
# Auto-reload when secrets change (checks every 10s by default)
infisical run --watch -- nodemon src/server.js

# Faster watch interval
infisical run --watch --watch-interval=5 -- nodemon src/server.js
```

### CI/CD

```bash
# Service token
export INFISICAL_TOKEN=<service-token>
infisical run --env=production -- npm test

# Inline token
infisical run --token=<token> --env=production -- npm deploy
```

### Override secrets

```bash
# Override specific secrets (overrides take precedence)
infisical run --env=production --secret-overriding API_KEY=local-dev-key -- node app.js
```

## How It Works

1. Infisical fetches secrets for the specified `--env` and `--path`
2. Secrets are set as environment variables (`SECRET_NAME` → `SECRET_NAME`)
3. Your command runs with those env vars available
4. Child processes inherit all injected environment variables

## Notes

- Run from your project directory (where `.infisical.json` is) or use `--project-config-dir`
- Secrets are **never printed to stdout** except with `--silent` for CLI metadata
- Override values take precedence over fetched secrets
- **`--watch` is ideal for development** — restarting apps when secrets change
- Use `--recursive` to include secrets from all nested folder paths