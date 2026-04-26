# infisical-tokens

Token management for Infisical CLI — service tokens and personal access tokens.

## Triggers

- `infisical token`
- `infisical service-token`
- `service token`
- `access token`

## Commands

### `token`

Personal access token management.

### `service-token`

Service token lifecycle management.

#### Sub-commands

| Command | Description |
|---------|-------------|
| `create` | Create a new service token |
| `list` | List service tokens for project/org |
| `delete` | Delete a service token |

#### Flags

| Flag | Description | Required |
|------|-------------|----------|
| `--projectId` | Project ID scope | Yes |
| `--organizationId` | Organization ID scope | Yes |
| `--name` | Token name/label | Yes |
| `--env` | Environment (e.g., `dev`, `prod`) | No |
| `--expires-at` | Expiration timestamp | No |
| `--scope` | Token scope/permissions | No |

## Examples

### Create a service token

```bash
infisical service-token create \
  --projectId="proj_xxxxxxxxxxxx" \
  --organizationId="org_xxxxxxxxxxxx" \
  --name="CI/CD Token" \
  --env="dev" \
  --expires-at="2025-12-31T23:59:59Z"
```

### List service tokens

```bash
infisical service-token list --projectId="proj_xxxxxxxxxxxx"
```

### Delete a service token

```bash
infisical service-token delete --projectId="proj_xxxxxxxxxxxx" --name="CI/CD Token"
```