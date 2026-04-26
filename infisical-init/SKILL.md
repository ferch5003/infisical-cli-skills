---
name: infisical-init
description: Initialize Infisical in a project. Use when user mentions infisical init, initialize project, or connect project to Infisical.
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
        scope: project lookup and configuration
---

# infisical-init

Initialize Infisical in a project.

## Command

```bash
infisical init
```

## Description

Connects your local project to an Infisical project by creating a `.infisical.json` configuration file in the working directory. This file links your project to an Infisical project and environment.

**Note:** This command is **fully interactive**. It requires terminal interaction to select organization, project, and environment from a picker. There are no non-interactive flags.

## Prerequisite

Requires authentication. Run `infisical login` first if not already authenticated.

```bash
infisical login
infisical init
```

## Interactive Flow

When you run `infisical init`, it will:

1. Prompt you to select an **organization**
2. Prompt you to select a **project** within that organization
3. Prompt you to select an **environment** (e.g., development, staging, production)
4. Create the `.infisical.json` file

## Output

Creates `.infisical.json` in the project root:

```json
{
  "projectId": "<project-id>",
  "env": "<environment>"
}
```

## Usage

```bash
# Basic initialization (interactive)
infisical init
```

The command has **no non-interactive flags**. To use it in CI/CD or scripts, consider:

```bash
# Option 1: Pre-create .infisical.json manually
echo '{"projectId":"abc123","env":"production"}' > .infisical.json

# Option 2: Use INFISICAL_TOKEN env var to skip auth, then init
export INFISICAL_TOKEN=<your-token>
infisical init  # still interactive for project selection

# Option 3: Export from an existing project config
infisical secrets --env=production --export > .env
```

## Notes

- Run from the **root of your project**
- Requires authentication (`infisical login`)
- The config file **`.infisical.json`** can be committed to version control
- To change the project/environment, run `infisical init` again or edit `.infisical.json`
- If the project has already been initialized, it will ask if you want to reinitialize

## Common Workflows

### New project setup

```bash
infisical login
infisical init
# Select organization, project, and environment from the interactive picker
```

### Switch to a different environment

```bash
# Option A: Reinitialize
infisical init

# Option B: Edit the config directly
# Edit .infisical.json and change the "env" value
```

## Related Commands

| Command | Description |
|---------|-------------|
| `infisical secrets` | Manage secrets after initialization |
| `infisical login` | Authenticate first if needed |
| `infisical run` | Run commands with secrets injected |
| `infisical export` | Export secrets to a file |