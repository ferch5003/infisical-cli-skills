---
name: infisical-bootstrap
description: Bootstrap a self-hosted Infisical instance. Use when user mentions infisical bootstrap, setup Infisical instance, or initialize self-hosted.
metadata:
  openclaw:
    requires:
      bins: [infisical]
    network:
      - description: Outbound HTTPS to the self-hosted Infisical instance
        scope: instance bootstrap operations
---

# infisical-bootstrap

Bootstrap a **self-hosted Infisical instance**. This command is used when setting up a new Infisical instance on your own infrastructure — it is NOT for initializing a project.

**Note:** For project initialization, use `infisical init` instead.

## Command

```bash
infisical bootstrap
```

## Description

Creates the initial admin user and organization for a self-hosted Infisical instance. This is the first command to run after deploying Infisical via Docker, Kubernetes, or binary.

## Flags

| Flag | Description | Required |
|------|-------------|----------|
| `--domain` | Domain of your self-hosted Infisical instance | No |
| `--email` | Admin email address | Yes |
| `--password` | Admin password | Yes |
| `--organization` | Name of the organization to create | Yes |
| `--output` | Output format: `json` or `k8-secret` | No |
| `--ignore-if-bootstrapped` | Continue if instance is already bootstrapped | No |
| `--k8-secret-name` | Kubernetes secret name (k8-secret output) | No |
| `--k8-secret-namespace` | Kubernetes namespace (k8-secret output) | No |
| `--k8-secret-template` | Template for rendering the K8s secret | No |

## Usage

### Basic bootstrap (non-interactive)

```bash
infisical bootstrap \
  --domain https://infisical.internal.com \
  --email admin@example.com \
  --password "secure-password" \
  --organization "My Company"
```

### JSON output (for automation)

```bash
infisical bootstrap \
  --domain https://infisical.internal.com \
  --email admin@example.com \
  --password "secure-password" \
  --organization "My Company" \
  --output json
```

### Kubernetes secret output

```bash
infisical bootstrap \
  --domain https://infisical.internal.com \
  --email admin@example.com \
  --password "secure-password" \
  --organization "My Company" \
  --output k8-secret \
  --k8-secret-name infisical-admin \
  --k8-secret-namespace infisical
```

### Skip if already bootstrapped

```bash
infisical bootstrap \
  --domain https://infisical.internal.com \
  --email admin@example.com \
  --password "secure-password" \
  --organization "My Company" \
  --ignore-if-bootstrapped
```

## When to use

| Task | Command |
|------|---------|
| Set up a new self-hosted Infisical instance | `infisical bootstrap` |
| Connect a project to Infisical | `infisical init` |
| Authenticate with Infisical | `infisical login` |

## Notes

- This command is for **self-hosted instances only** — do not use with Infisical Cloud
- Requires the Infisical instance to be running and accessible
- The `--domain` flag should point to your self-hosted instance URL
- If `--ignore-if-bootstrapped` is not set and the instance is already initialized, the command will fail

## Related Commands

| Command | Description |
|---------|-------------|
| `infisical init` | Connect a project to Infisical |
| `infisical login` | Authenticate with Infisical |
| `infisical reset` | Reset local configuration |