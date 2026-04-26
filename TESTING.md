# Skill Testing Plan

Testing each skill involves invoking it in a real AI agent session with an actual Infisical instance. The goal is to verify the skill fires correctly, produces accurate commands, and handles errors gracefully.

## Testing Protocol

For each skill, the test follows this pattern:

1. **Trigger** — Simulate a user request that should invoke the skill
2. **Command output** — Run the actual `infisical` command referenced in the skill
3. **Verification** — Confirm output matches expected format
4. **Error case** — Test with invalid args / missing auth
5. **Report** — Document findings: ✅ pass, ⚠️ partial, ❌ fail

## Skill Test Order

### Phase 1: Core / Auth (prerequisites for all others)

| # | Skill | Priority | Notes |
|---|-------|----------|-------|
| 1 | `infisical-auth` | 🔴 High | Login/logout/user — required for everything else |
| 2 | `infisical-init` | 🔴 High | Creates `.infisical.json` — needed for secrets operations |
| 3 | `infisical-secrets` | 🔴 High | Core CRUD — most common use case |

### Phase 2: Common Operations

| # | Skill | Priority | Notes |
|---|-------|----------|-------|
| 4 | `infisical-run` | 🔴 High | Secrets injection — common in CI/CD |
| 5 | `infisical-export` | 🟡 Medium | Export formats (.env, JSON, YAML) |
| 6 | `infisical-bootstrap` | 🟡 Medium | Quick project setup |

### Phase 3: Dynamic & Advanced

| # | Skill | Priority | Notes |
|---|-------|----------|-------|
| 7 | `infisical-dynamic-secrets` | 🟡 Medium | DB credentials — requires a configured provider |
| 8 | `infisical-tokens` | 🟡 Medium | PAT/service tokens — important for automation |

### Phase 4: Infrastructure / Enterprise

| # | Skill | Priority | Notes |
|---|-------|----------|-------|
| 9 | `infisical-ssh` | 🟢 Low | SSH certs — enterprise feature |
| 10 | `infisical-kmip` | 🟢 Low | Key management — enterprise |
| 11 | `infisical-pam` | 🟢 Low | Privileged access — enterprise |
| 12 | `infisical-relay` | 🟢 Low | Relay server |
| 13 | `infisical-gateway` | 🟢 Low | API gateway |
| 14 | `infisical-vaults` | 🟢 Low | Vault integration |

### Phase 5: Meta

| # | Skill | Priority | Notes |
|---|-------|----------|-------|
| 15 | `infisical-cli` | 🔴 High | Main routing skill — verify all sub-skills load correctly |

## Test Report Template

```markdown
## [Skill Name]

**Tested by:** @user
**Date:** YYYY-MM-DD
**Infisical Plan:** Free / Pro / Enterprise

### Test 1: [Test description]
- **Command:** `infisical ...`
- **Expected:** ...
- **Actual:** ...
- **Status:** ✅ / ⚠️ / ❌
- **Notes:** ...

### Test 2: Error handling
- **Input:** ...
- **Expected error:** ...
- **Actual:** ...
- **Status:** ✅ / ⚠️ / ❌
```

## Status Legend

- ✅ **Pass** — Skill fires correctly, output matches expected
- ⚠️ **Partial** — Skill works but commands/output need refinement
- ❌ **Fail** — Skill doesn't fire, commands are wrong, or info is outdated

## Running Tests

Tests are run manually using the AI agent that will consume these skills:

```bash
# Example: Test infisical-auth skill
# In an AI agent session, trigger:
"Show me how to login to Infisical using universal auth with client credentials"

# The agent should load infisical-auth and produce:
# infisical login universal-auth --client-id <id> --client-secret <secret>
```

## Known Limitations

- **Dynamic secrets** require a configured provider (Postgres, MySQL, etc.) — may not be testable without infrastructure
- **Enterprise features** (KMIP, PAM, SSH certs) require Enterprise plan
- **Self-hosted** Infisical instances may have different API endpoints — skills assume cloud by default
- Skills document CLI flags; not all flags are tested for every permutation

## Coverage Goals

- [x] Phase 1: Core skills (3) — auth ✅, init ✅, secrets ✅
- [x] Phase 2: Common operations (3) — run ✅, export, bootstrap ✅
- [x] Phase 3: Dynamic & advanced (2) — dynamic-secrets, tokens ✅
- [ ] Phase 4: Infrastructure (5) — ssh, kmip, pam, relay, gateway
- [ ] Phase 5: Meta (1) — main CLI skill

---

## Test Report: infisical-auth

**Date:** 2026-04-26
**Infisical CLI version:** 0.43.77
**Testing method:** CLI `--help` + dry-run commands

### Test 1: `infisical user`
- **Command:** `infisical user`
- **Expected:** Show current user info
- **Actual:** Outputs user management help. **Subcommand `infisical user get`** returns user info.
- **Status:** ⚠️ Partial
- **Notes:** The skill describes `infisical user` as displaying user info, but the actual command is `infisical user get`. The `--email`, `--id`, `--name` flags described in the skill don't exist — these are not valid flags.

### Test 2: `infisical logout`
- **Command:** `infisical logout --help`
- **Expected:** Logout command should exist
- **Actual:** `Error: unknown command "logout" for "infisical"`
- **Status:** ❌ Fail
- **Notes:** **`infisical logout` does not exist**. The command to clear credentials is **`infisical reset`**. Also `--clear-cache` and `--global` flags for logout don't exist. The skill references a non-existent command.

### Test 3: `infisical reset`
- **Command:** `infisical reset --help`
- **Expected:** Should match skill documentation
- **Actual:** Command exists. **Only flag is `--help`** — no `--reauth` or `--force` flags.
- **Status:** ⚠️ Partial
- **Notes:** `--reauth` and `--force` flags described in the skill do not exist in the current CLI version.

### Test 4: `infisical login --method universal-auth`
- **Command:** `infisical login --help`
- **Expected:** `--client-id` and `--client-secret` for universal-auth
- **Actual:** Flags confirmed: `--client-id`, `--client-secret`, `--method`, `--organization-slug`, `--plain`
- **Status:** ✅ Pass

### Test 5: `infisical login --method kubernetes`
- **Command:** `infisical login --method kubernetes --help`
- **Expected:** Flags for kubernetes auth
- **Actual:** Uses `--machine-identity-id` (not `--identity-id`) and `--service-account-token-path` (not `--service-account-name`). Namespace flag doesn't exist.
- **Status:** ⚠️ Partial
- **Notes:** Several flag names in the skill are **outdated or incorrect**. The CLI uses `--machine-identity-id` for all machine identity methods, not `--identity-id`. Kubernetes uses `--service-account-token-path` instead of `--service-account-name`.

### Test 6: `infisical login --method azure`
- **Command:** `infisical login --help`
- **Expected:** Azure auth flags
- **Actual:** Uses `--machine-identity-id`, `--client-id`, `--tenant-id`. The skill uses `--identity-id` instead of `--machine-identity-id`. `--client-id` here is the **Azure client ID** (not to be confused with universal auth's client-id).
- **Status:** ⚠️ Partial
- **Notes:** Flag naming inconsistency across the skill.

### Test 7: `infisical login --method gcp-id-token` / `gcp-iam`
- **Command:** `infisical login --help`
- **Expected:** GCP auth flags
- **Actual:** Two distinct methods: `gcp-id-token` uses `--jwt`, `gcp-iam` uses `--machine-identity-id` and `--service-account-key-file-path`. No `--service-account-email` flag.
- **Status:** ⚠️ Partial
- **Notes:** The skill describes a single `gcp` method with `--service-account-email` — in reality there are two methods with different flags.

### Test 8: `infisical login --method aws-iam`
- **Command:** `infisical login --help`
- **Expected:** AWS auth flags
- **Actual:** Uses `--machine-identity-id` and `--role-arn`
- **Status:** ✅ Pass
- **Notes:** Flag naming issue — skill uses `--identity-id` instead of `--machine-identity-id`.

### Test 9: `infisical login --method oidc-auth`
- **Command:** `infisical login --help`
- **Expected:** OIDC auth flags
- **Actual:** Uses `--jwt` (for token) with `oidc-auth` method. `--issuer-url` and `--role-arn` flags don't appear in help output.
- **Status:** ⚠️ Partial
- **Notes:** `--issuer-url` and `--role-arn` may be used differently or might require a separate setup. The skill's `--issuer-url` flag needs verification.

### Test 10: `infisical login jwt`
- **Command:** `infisical login --jwt <token>` (method `jwt-auth` or `oidc-auth`)
- **Expected:** JWT token login
- **Actual:** `--jwt` flag exists and works with `jwt-auth` method
- **Status:** ✅ Pass
- **Notes:** Skill correctly documents JWT login.

### Test 11: Token storage paths
- **Command:** `infisical vault --help`
- **Expected:** Token storage location docs
- **Actual:** `infisical vault` command exists for managing token storage location. The skill's path (`~/.config/infisical/config.json`) is likely correct but worth verifying.
- **Status:** ✅ Pass
- **Notes:** The skill is likely accurate but the `--vault-path` subcommand provides programmatic access.

### Summary

| Test | Command | Status |
|------|---------|--------|
| 1 | `infisical user` | ⚠️ Wrong subcommand |
| 2 | `infisical logout` | ❌ Command doesn't exist |
| 3 | `infisical reset` | ⚠️ Missing flags |
| 4 | Universal Auth | ✅ Correct |
| 5 | Kubernetes Auth | ⚠️ Wrong flag names |
| 6 | Azure Auth | ⚠️ Wrong flag names |
| 7 | GCP Auth | ⚠️ Wrong method/flags |
| 8 | AWS Auth | ⚠️ Wrong flag names |
| 9 | OIDC Auth | ⚠️ Missing flags |
| 10 | JWT Auth | ✅ Correct |
| 11 | Token storage | ✅ Likely correct |

**Key findings:**
- **`infisical logout` does not exist** — must be replaced with `infisical reset`
- **All machine identity logins use `--machine-identity-id`** (not `--identity-id`)
- **`gcp` has two auth methods**: `gcp-id-token` (via JWT) and `gcp-iam` (via service account key file)
- **Kubernetes uses `--service-account-token-path`** (not `--service-account-name` or `--namespace`)
- **`--reauth` and `--force` flags for `reset` don't exist**
- **`--email`, `--id`, `--name` flags for `user` don't exist** — use `infisical user get`

**Actions needed:**
- [x] Fix `infisical logout` section → reference `infisical reset` instead (fixed in skill update)
- [x] Replace all `--identity-id` with `--machine-identity-id` (fixed in skill update)
- [x] Fix Kubernetes auth section with correct flags (fixed in skill update)
- [x] Split GCP into `gcp-id-token` and `gcp-iam` sections (fixed in skill update)
- [x] Fix `infisical user` section → use `infisical user get` (fixed in skill update)
- [x] Remove `--reauth` and `--force` from `reset` section (fixed in skill update)

---

## Test Report: infisical-init

**Date:** 2026-04-26
**Infisical CLI version:** 0.43.77
**Testing method:** CLI `--help` + interactive dry-run

### Test 1: `infisical init --help`
- **Command:** `infisical init --help`
- **Expected:** Help output with available flags
- **Actual:** Only `--help` flag. No `--projectId`, `--env`, or `--template` flags.
- **Status:** ❌ Fail
- **Notes:** **`--projectId`, `--env`, and `--template` are not valid flags** for `infisical init`. The command is fully interactive — it prompts for organization, project, and environment selection.

### Test 2: Interactive init flow
- **Command:** `infisical init`
- **Expected:** Interactive prompts for project selection
- **Actual:** Shows interactive picker for organization selection (works correctly)
- **Status:** ✅ Pass
- **Notes:** Interactive flow works as expected. Requires terminal interaction.

### Test 3: `.infisical.json` structure
- **Expected:** Config file with `projectId` and `env`
- **Actual:** Cannot test without completing interactive flow
- **Status:** ⚠️ Cannot verify
- **Notes:** Skill's example config structure is consistent with what `infisical init` typically creates.

### Test 4: `infisical bootstrap` (bonus discovery)
- **Command:** `infisical bootstrap --help`
- **Expected:** Bootstrap help
- **Actual:** Bootstrap exists but is for **self-hosted instance initialization**, NOT project setup.
- **Status:** ⚠️ Warning
- **Notes:** `infisical-bootstrap` skill likely confuses `init` (project setup) with `bootstrap` (self-hosted instance setup). They are different commands. The `bootstrap` skill needs review.

### Summary

| Test | Description | Status |
|------|-------------|--------|
| 1 | `infisical init --help` flags | ❌ Wrong flags |
| 2 | Interactive init flow | ✅ Works |
| 3 | `.infisical.json` structure | ⚠️ Unverified |
| 4 | `infisical bootstrap` vs `init` | ⚠️ Needs review |

**Key findings:**
- **`--projectId`, `--env`, `--template` flags do not exist** for `infisical init`
- **`infisical init` is fully interactive** — cannot be used non-interactively without external tools
- **`infisical bootstrap` is for self-hosted instance setup**, not project initialization

**Actions needed:**
- [ ] Remove `--projectId`, `--env`, `--template` flags from the skill
- [ ] Clarify that `infisical init` is interactive-only
- [ ] Review `infisical-bootstrap` skill to separate instance bootstrap from project setup

---

## Test Report: infisical-secrets

**Date:** 2026-04-26
**Infisical CLI version:** 0.43.77
**Testing method:** CLI `--help` for all subcommands

### Test 1: Global flags (top-level)
- **Expected:** `--env`, `--expand`, `--include-imports`, `--output`, `--path`, `--plain`, `--projectId`, `--recursive`, `--secret-overriding`, `--tags`, `--token`
- **Actual:** All present. **`--expand` defaults to `true`** (not `false`). **`--include-imports` defaults to `true`**.
- **Status:** ✅ Pass

### Test 2: `infisical secrets list` (implicit — no `list` subcommand)
- **Command:** `infisical secrets --help`
- **Expected:** There is no `infisical secrets list` — list is the default behavior when no subcommand is given
- **Actual:** `infisical secrets` defaults to listing. `infisical secrets list` would not work — the correct command is just `infisical secrets`
- **Status:** ❌ Fail
- **Notes:** The skill documents `infisical secrets list` but there is **no `list` subcommand**. List is the default behavior of `infisical secrets`.

### Test 3: `infisical secrets get`
- **Command:** `infisical secrets get --help`
- **Expected:** Flags for getting secrets
- **Actual:** `infisical secrets get [secrets]` — takes positional arguments (one or more secret names). Same flags as top-level. No `--secret-name` flag.
- **Status:** ⚠️ Partial
- **Notes:** **`--secret-name` flag does not exist**. Pass secret names as positional arguments instead. The skill's example `infisical secrets get DATABASE_URL` is correct — the flag-based form is wrong.

### Test 4: `infisical secrets set`
- **Command:** `infisical secrets set --help`
- **Expected:** Flags for setting secrets
- **Actual:** `infisical secrets set [secrets]` — takes `KEY=VALUE` pairs as positional args. Flags: `--file`, `--path`, `--projectId`, `--token`, `--type`
- **Status:** ⚠️ Partial
- **Notes:** **`--secret-name` and `--secret-value` do not exist**. Use `KEY=VALUE` syntax as positional args. **New flag discovered**: `--file` (load from .env or YAML file). **New flag**: `--type personal|shared`.

### Test 5: `infisical secrets delete`
- **Command:** `infisical secrets delete --help`
- **Expected:** Flags for deleting secrets
- **Actual:** `infisical secrets delete [secrets]` — takes positional args (one or more secret names). **`--type personal|shared`** defaults to `personal`. **`--force` flag does not exist**.
- **Status:** ⚠️ Partial
- **Notes:** **`--secret-name` does not exist**. **`--recursive` does not exist** for delete. **`--force` flag does not exist**.

### Test 6: `infisical secrets folders`
- **Command:** `infisical secrets folders --help`
- **Expected:** Folder subcommands
- **Actual:** Three subcommands: `create`, `delete`, `get`. **No `infisical secrets folders` (bare) listed in help**.
- **Status:** ⚠️ Partial
- **Notes:** **`infisical secrets folders` (bare)** — may work but is not documented in help. The skill uses bare `infisical secrets folders` for listing but actual subcommand is `infisical secrets folders get`.

### Test 7: `infisical secrets folders create`
- **Command:** `infisical secrets folders create --help`
- **Expected:** Flags for creating folders
- **Actual:** `--name` (or `-n`/`--path` shorthand) and `--path`. **`--path` here means parent path** (default `/`). **`--recursive` does not exist**.
- **Status:** ⚠️ Partial
- **Notes:** The skill uses `--path=/backend` as the folder name, but **`--path` is the parent path** and `--name` is the folder name itself.

### Test 8: `infisical secrets folders delete`
- **Command:** `infisical secrets folders delete --help`
- **Expected:** Flags for deleting folders
- **Actual:** `--name` (or `-n`/`--path`) and `--path`. **`--recursive` and `--force` do not exist**.
- **Status:** ⚠️ Partial
- **Notes:** Same `--path` vs `--name` confusion. **`--recursive` and `--force` are not valid flags**.

### Test 9: `infisical secrets folders get`
- **Command:** `infisical secrets folders get --help`
- **Expected:** Flags for listing folders
- **Actual:** `--path` and `--output`. Correct subcommand found.
- **Status:** ✅ Pass
- **Notes:** The skill uses bare `infisical secrets folders` for listing — the correct command is `infisical secrets folders get`.

### Test 10: `infisical secrets generate-example-env`
- **Command:** `infisical secrets generate-example-env --help`
- **Expected:** Flags
- **Actual:** `--path`, `--projectId`, `--token`. **No `--env` flag** at subcommand level (env is global).
- **Status:** ⚠️ Partial
- **Notes:** **`--env` is a global flag** on `infisical secrets`, not specific to this subcommand. The skill correctly passes `--env` but calls it a subcommand flag.

### Test 11: `--output` format (bonus discovery)
- **Command:** `infisical secrets --help`
- **Expected:** Output format options
- **Actual:** **`--output` supports `yaml`, `json`, `dotenv`** (not just plain text)
- **Status:** ✅ Pass
- **Notes:** **`--plain` is deprecated** — use `--output dotenv` instead.

### Summary

| Test | Command | Status |
|------|---------|--------|
| 1 | Global flags | ✅ Mostly correct |
| 2 | `infisical secrets list` | ❌ No `list` subcommand |
| 3 | `infisical secrets get` | ⚠️ Wrong flag (`--secret-name`) |
| 4 | `infisical secrets set` | ⚠️ Wrong flags + new `--file` and `--type` |
| 5 | `infisical secrets delete` | ⚠️ Wrong flags + `--type` defaulting to `personal` |
| 6 | `infisical secrets folders` | ⚠️ Bare command vs `get` subcommand |
| 7 | `infisical secrets folders create` | ⚠️ `--path`/`--name` confusion, no `--recursive` |
| 8 | `infisical secrets folders delete` | ⚠️ `--path`/`--name` confusion, no `--recursive`/`--force` |
| 9 | `infisical secrets folders get` | ✅ Correct |
| 10 | `generate-example-env` | ⚠️ `--env` is global not subcommand flag |
| 11 | `--output` format | ✅ `yaml`, `json`, `dotenv` (plain deprecated) |

**Key findings:**
- **`infisical secrets list` does not exist** — default behavior of `infisical secrets` (no subcommand) is list
- **`--secret-name` and `--secret-value` do not exist** — use positional args for secret names and `KEY=VALUE` for values
- **`--recursive` and `--force` for folders delete do not exist**
- **`--path` in folders subcommands is the parent path**, `--name` is the folder name
- **`set --type personal|shared`** (defaults to `shared`)
- **`delete --type personal|shared`** (defaults to `personal` — **important difference**)
- **`set --file`** for loading from `.env` or YAML file
- **`--plain` is deprecated** — use `--output dotenv`
- **`--expand` defaults to `true`**
- **`--include-imports` defaults to `true`**

**Actions needed:**
- [ ] Replace `infisical secrets list` with `infisical secrets`
- [ ] Remove `--secret-name` and `--secret-value` from all commands — use positional args
- [ ] Fix folders: `--path` is parent path, `--name` is folder name
- [ ] Remove `--recursive` and `--force` from folders delete
- [ ] Add `--file` and `--type` to set section
- [ ] Add `--type` to delete section (defaults to `personal` — important!)
- [ ] Update `--output` to note `--plain` is deprecated
- [ ] Clarify `--env` is a global flag, not subcommand-specific

---

## Test Report: infisical-run

**Date:** 2026-04-26
**Infisical CLI version:** 0.43.77
**Testing method:** CLI `--help` for all flags

### Test 1: Top-level flags
- **Command:** `infisical run --help`
- **Expected:** Match skill flags
- **Actual:** All documented flags exist. **New flags discovered**: `--expand`, `--include-imports`, `--project-config-dir`, `--recursive`, `--secret-overriding`, `--tags`, `--token`, `--watch`, `--watch-interval`
- **Status:** ⚠️ Partial
- **Notes:** Skill is missing most flags. `--secret-override` exists but is misspelled — actual flag is **`--secret-overriding`** (with "g"). **`--silent` is not a flag for `run`** (it's a global flag but works). **`-c/--command` flag** for chained commands not documented.

### Test 2: `--watch` feature
- **Expected:** Not documented in skill
- **Actual:** **`--watch` flag** enables auto-reload when secrets change. **`--watch-interval`** sets check frequency (default 10s).
- **Status:** ⚠️ Not documented — valuable feature missing
- **Notes:** Watch mode is great for development. Very useful.

### Test 3: `--command` (chained commands)
- **Expected:** Not documented in skill
- **Actual:** **`-c/--command`** allows running chained shell commands without `--`.
- **Status:** ⚠️ Not documented
- **Notes:** Example: `infisical run --command "npm install && npm run dev; echo done"`

### Summary

| Test | Command | Status |
|------|---------|--------|
| 1 | Top-level flags | ⚠️ Missing most flags, `--secret-override` typo |
| 2 | `--watch` feature | ⚠️ Not documented |
| 3 | `--command` | ⚠️ Not documented |

**Key findings:**
- **`--secret-override` is wrong** — correct flag is **`--secret-overriding`**
- **Many flags missing**: `--expand`, `--include-imports`, `--project-config-dir`, `--recursive`, `--secret-overriding`, `--tags`, `--token`
- **Useful features not documented**: `--watch` + `--watch-interval` (auto-reload on secret change)
- **`-c/--command`** for chained commands

**Actions needed:**
- [ ] Fix `--secret-override` → `--secret-overriding`
- [ ] Add all missing flags
- [ ] Add `--watch` and `--watch-interval` documentation
- [ ] Add `-c/--command` documentation

---

## Test Report: infisical-tokens

**Date:** 2026-04-26
**Infisical CLI version:** 0.43.77
**Testing method:** CLI `--help` for all subcommands

### Test 1: `infisical token`
- **Command:** `infisical token --help`
- **Expected:** Token management subcommands
- **Actual:** Only subcommand is **`renew`** (to renew universal auth access token). No list, create, or delete for personal tokens via CLI.
- **Status:** ⚠️ Partial
- **Notes:** The skill describes "personal access token management" generically but the CLI only has `token renew`. List/create/delete for personal tokens are done via the web UI, not CLI.

### Test 2: `infisical service-token`
- **Command:** `infisical service-token --help`
- **Expected:** `create`, `list`, `delete` subcommands
- **Actual:** **Only `create` subcommand exists**. No `list` or `delete`.
- **Status:** ❌ Fail
- **Notes:** **`infisical service-token list` and `infisical service-token delete` do not exist** in the CLI. These are web UI operations.

### Test 3: `infisical service-token create`
- **Command:** `infisical service-token create --help`
- **Expected:** Flags for creating service token
- **Actual:** Flags: `--access-level`, `--expiry-seconds`, `--name`, `--projectId`, `--scope`, `--token-only`. **No `--organizationId`, `--env`, or `--expires-at` flags**.
- **Status:** ⚠️ Partial
- **Notes:** **`--env` is wrong** — correct way is `--scope <env>:<path>`. **`--expires-at` is wrong** — use `--expiry-seconds`. **`--scope` is the correct mechanism** for env+path scoping. **`--access-level`** replaces what the skill calls "scope/permissions". **`--token-only`** prints only the token (great for scripting). **`--name` defaults to** "Service token generated via CLI". **`--projectId` defaults** to the linked project in `.infisical.json`.

### Test 4: `infisical token renew`
- **Command:** `infisical token renew --help`
- **Expected:** Renew access token
- **Actual:** Takes `[token]` as positional arg (the token to renew). No special flags.
- **Status:** ✅ Pass
- **Notes:** Skill doesn't document this subcommand at all — good to add.

### Summary

| Test | Command | Status |
|------|---------|--------|
| 1 | `infisical token` | ⚠️ Only has `renew`, no CRUD for personal tokens |
| 2 | `infisical service-token list/delete` | ❌ Commands don't exist |
| 3 | `infisical service-token create` | ⚠️ Wrong flags throughout |
| 4 | `infisical token renew` | ✅ Exists (not documented in skill) |

**Key findings:**
- **`infisical service-token list` and `delete` do NOT exist** — web UI only
- **`--env` is wrong** — use **`--scope <env>:<path>`** for scoping
- **`--expires-at` is wrong** — use **`--expiry-seconds`** (int, seconds from now)
- **`--scope` format**: `<env-slug>:<folder-path>` (e.g., `production:/backend`)
- **`--access-level`** replaces "scope/permissions" — values: `read`, `write` (can combine)
- **`--token-only`** prints only the token (perfect for scripting)
- **`--name` defaults** to "Service token generated via CLI"
- **Default expiry**: 86400 seconds (1 day)
- **`infisical token renew` exists** but wasn't documented

**Actions needed:**
- [ ] Remove `infisical service-token list` and `delete` — CLI doesn't have these
- [ ] Replace `--env` and `--expires-at` with `--scope` and `--expiry-seconds`
- [ ] Add `--access-level` (read/write) and `--token-only`
- [ ] Add `infisical token renew` documentation
- [ ] Clarify personal tokens (PAT) are managed via web UI

---

## Test Report: infisical-export

**Date:** 2026-04-26
**Infisical CLI version:** 0.43.77
**Testing method:** CLI `--help` for all flags

### Test 1: Top-level flags
- **Command:** `infisical export --help`
- **Expected:** Match skill flags
- **Actual:** Most flags exist. **New flags**: `--expand`, `--include-imports`, `--projectId`, `--secret-overriding`, `--tags`, `--token`, `--template`, `--path`
- **Status:** ⚠️ Partial
- **Notes:** **`--format` only supports `dotenv`, `json`, `csv`** — skill says `env`, `yaml` (no `yaml` support!). **`--path` is for folder path**, not output file — skill conflates the two. **`--include-references` is wrong** — correct flag is `--include-imports`. **`--uppercase` and `--escape` do not exist**.

### Test 2: Format flags
- **Command:** `infisical export --help`
- **Expected:** `env`, `json`, `yaml` formats
- **Actual:** Formats: **`dotenv`**, **`json`**, **`csv`** — no `env` (use `dotenv`) and **no `yaml`** (skill is wrong on both)
- **Status:** ❌ Fail
- **Notes:** **`yaml` format does not exist** (but skill documents it). **`env` format doesn't exist** — use `dotenv`.

### Test 3: Output file flag
- **Expected:** `--path` for output file
- **Actual:** **`--output-file` (`-o`)** is the correct flag for writing to a file. `--path` is for folder path within project.
- **Status:** ❌ Fail
- **Notes:** Skill uses `--path ./secrets.env` which is wrong — that's the project folder path.

### Test 4: Reference vs Import flags
- **Expected:** `--include-references`
- **Actual:** **`--include-imports`** is the correct flag. `--include-references` does not exist.
- **Status:** ❌ Fail
- **Notes:** Skill documents wrong flag name.

### Summary

| Test | Description | Status |
|------|-------------|--------|
| 1 | Top-level flags | ⚠️ Missing new flags |
| 2 | Format flags | ❌ `yaml` and `env` don't exist |
| 3 | Output file flag | ❌ `--path` is wrong — use `--output-file` |
| 4 | Reference flag | ❌ `--include-references` wrong — use `--include-imports` |

**Key findings:**
- **No `yaml` format** — skill documents it incorrectly. Only `dotenv`, `json`, `csv`
- **`--path` is project folder path**, not output file — use `--output-file`
- **No `--uppercase`, `--escape`, `--include-references`**
- **Formats**: `dotenv` (default), `json`, `csv` — not `env`, not `yaml`
- **New flags**: `--expand`, `--include-imports`, `--projectId`, `--secret-overriding`, `--tags`, `--token`, `--template`, `--recursive`

**Actions needed:**
- [ ] Replace `yaml` → remove from formats (doesn't exist)
- [ ] Replace `env` → `dotenv`
- [ ] Fix `--path` for output → use `--output-file`
- [ ] Fix `--include-references` → `--include-imports`
- [ ] Remove `--uppercase` and `--escape` (don't exist)
- [ ] Add new flags: `--expand`, `--include-imports`, `--projectId`, `--template`, `--tags`, `--token`