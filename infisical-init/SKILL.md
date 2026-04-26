# infisical-init

Initialize Infisical in a project.

## Triggers

- `infisical init`
- `initialize project`

## Command

`init`

## Description

Initializes Infisical in your project by creating a `.infisical.json` configuration file in the working directory. This file links your local project to an Infisical project and environment.

## Creates

- `.infisical.json` — Project configuration file in the working directory

## Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--projectId` | Specify the Infisical project ID | `--projectId abc123` |
| `--env` | Specify the environment name | `--env development` |
| `--template` | Initialize from a template | `--template my-template` |

## Usage

```bash
# Basic initialization
infisical init

# With project ID
infisical init --projectId abc123

# With environment
infisical init --env production

# Combine flags
infisical init --projectId abc123 --env staging
```

## Output

Creates `.infisical.json`:

```json
{
  "projectId": "<project-id>",
  "env": "<environment>"
}
```

## Notes

- Run from the root of your project
- Requires authentication (run `infisical login` first if needed)
- The config file should be committed to version control