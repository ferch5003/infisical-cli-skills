# Infisical PAM Skill

## Metadata
- **name**: infisical-pam
- **Description**: Privileged Access Management operations
- **Triggers**: infisical pam, privileged access, pam

## Commands

| Command | Description |
|---------|-------------|
| `infisical pam list` | List privileged access requests |
| `infisical pam access-request` | Request elevated access |
| `infisical pam approve` | Approve access request |
| `infisical pam revoke` | Revoke active access |

## Examples
```bash
infisical pam list
infisical pam access-request --reason="Database maintenance" --duration=1h
infisical pam approve --request-id=abc123
infisical pam revoke --request-id=abc123
```
<!-- OMO_INTERNAL_INITIATOR -->