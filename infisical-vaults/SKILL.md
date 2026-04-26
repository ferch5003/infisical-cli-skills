# infisical-vaults

Vault management for Infisical CLI — create, list, update, and delete vaults.

## Triggers

- `infisical vault`
- `vault management`
- `infisical vault list`
- `infisical vault create`

## Commands

### `vault`

Vault lifecycle management.

#### Sub-commands

| Command | Description |
|---------|-------------|
| `list` | List vaults for project/organization |
| `create` | Create a new vault |
| `delete` | Delete a vault |
| `update` | Update vault settings |

#### Flags

| Flag | Description | Required |
|------|-------------|----------|
| `--projectId` | Project ID scope | Yes |
| `--organizationId` | Organization ID scope | Yes |
| `--name` | Vault name | Yes |
| `--description` | Vault description | No |
| `--environment` | Default environment | No |

## Examples

### List vaults

```bash
infisical vault list --projectId="proj_xxxxxxxxxxxx"
```

### Create a vault

```bash
infisical vault create \
  --projectId="proj_xxxxxxxxxxxx" \
  --organizationId="org_xxxxxxxxxxxx" \
  --name="Production Vault" \
  --description="Vault for production secrets"
```

### Delete a vault

```bash
infisical vault delete \
  --projectId="proj_xxxxxxxxxxxx" \
  --name="Production Vault"
```

### Update a vault

```bash
infisical vault update \
  --projectId="proj_xxxxxxxxxxxx" \
  --name="Production Vault" \
  --description="Updated description"
```