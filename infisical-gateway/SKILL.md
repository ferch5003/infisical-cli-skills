# Infisical Gateway Skill

## Metadata
- **name**: infisical-gateway
- **Description**: API Gateway configuration and management
- **Triggers**: infisical gateway, api gateway, gateway config

## Commands

| Command | Description |
|---------|-------------|
| `infisical gateway configure` | Configure gateway settings |
| `infisical gateway status` | Check gateway status |
| `infisical gateway routes` | List routing rules |
| `infisical gateway certificates` | Manage TLS certificates |

## Examples

```bash
# Configure gateway
infisical gateway configure --port=8080 --tls

# Check status
infisical gateway status

# List routes
infisical gateway routes --env=production

# Manage certificates
infisical gateway certificates list
```
