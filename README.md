# infisical-cli-skills

A collection of AI agent skills for the [Infisical CLI](https://infisical.com/docs/cli/overview) — designed to help AI coding agents (like Claude Code, OpenCode, etc.) interact with Infisical for secrets management, authentication, dynamic secrets, SSH certificates, and more.

## What is this?

When AI coding agents need to work with Infisical, they often lack knowledge of the Infisical CLI commands, flags, workflows, and best practices. This project ships modular, ready-to-use skills that give agents the domain knowledge to:

- Authenticate with Infisical (browser login, universal auth, cloud identity)
- Manage secrets (CRUD, environments, paths, export)
- Generate dynamic database/cloud credentials
- Handle SSH certificates, KMIP, PAM, and more

Each skill is self-contained and maps to a specific Infisical CLI domain.

## Skills

| Skill | Domain |
|-------|--------|
| `infisical-auth` | Login, logout, authentication methods (browser, universal, GCP, AWS, Azure, OIDC, JWT) |
| `infisical-secrets` | CRUD operations for secrets (list, get, set, delete, folders, export) |
| `infisical-dynamic-secrets` | Dynamic credentials for PostgreSQL, MySQL, MongoDB, Redis, AWS RDS, and more |
| `infisical-run` | Run commands with secrets injected into the environment |
| `infisical-init` | Project initialization (creates `.infisical.json`) |
| `infisical-bootstrap` | Quick project bootstrap and setup |
| `infisical-export` | Export secrets to various formats (`.env`, JSON, YAML) |
| `infisical-ssh` | SSH certificate management and signing |
| `infisical-pam` | Privileged Access Management operations |
| `infisical-kmip` | KMIP server operations for key management |
| `infisical-relay` | Relay server management |
| `infisical-gateway` | API gateway configuration |
| `infisical-tokens` | Personal access tokens and service tokens |
| `infisical-vaults` | Vault integration and management |
| `infisical-cli` | General CLI reference, quick start, and multi-domain routing |

## Installation

### For Claude Code

```bash
# Clone the repository into your Claude Code skills directory
git clone https://github.com/ferch5003/infisical-cli-skills.git ~/.claude/skills/infisical-cli
```

### For OpenCode

```bash
# Clone the repository into your OpenCode skills directory
git clone https://github.com/ferch5003/infisical-cli-skills.git ~/.agents/skills/infisical-cli
```

## Quick Start

```bash
# Authenticate
infisical login

# Initialize a project
infisical init

# List secrets
infisical secrets list

# Run with secrets injected
infisical run -- node app.js

# Generate dynamic credentials
infisical dynamic-secrets generate --provider postgres --secret-name db-creds
```

## Project Structure

```
infisical-cli-skills/
├── SKILL.md                      # Main skill (routes to sub-skills)
├── infisical-auth/              # Authentication commands
├── infisical-secrets/           # Secrets CRUD
├── infisical-dynamic-secrets/   # Dynamic credentials
├── infisical-run/               # Secrets injection
├── infisical-init/              # Project initialization
├── infisical-bootstrap/         # Project bootstrap
├── infisical-export/            # Export secrets
├── infisical-ssh/               # SSH certificates
├── infisical-pam/               # Privileged Access Management
├── infisical-kmip/              # KMIP key management
├── infisical-relay/             # Relay server
├── infisical-gateway/           # API gateway
├── infisical-tokens/            # Token management
└── infisical-vaults/            # Vault integration
```

## Skill Format

Each skill is a YAML + Markdown file with:

- **Frontmatter**: Metadata (name, description, triggers, dependencies, credentials)
- **Body**: Command reference, workflows, decision trees, examples

Agents load skills via their respective skill loaders and use the content to handle user requests involving Infisical CLI commands.

## Testing

See [TESTING.md](./TESTING.md) for the skill testing plan. Each skill is tested by:

1. **Manual invocation** — using the skill with a real Infisical instance
2. **Output verification** — confirming command output matches expected format
3. **Error handling** — testing edge cases (invalid auth, missing env, wrong flags)

## Contributing

Contributions welcome! To add or improve a skill:

1. Fork the repo
2. Add or update the skill file in the appropriate subdirectory
3. Ensure frontmatter metadata is complete
4. Test with a real Infisical environment
5. Open a PR

## License

MIT
