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

- [ ] Phase 1: Core skills (3) — auth, init, secrets
- [ ] Phase 2: Common operations (3) — run, export, bootstrap
- [ ] Phase 3: Dynamic & advanced (2) — dynamic-secrets, tokens
- [ ] Phase 4: Infrastructure (5) — ssh, kmip, pam, relay, gateway
- [ ] Phase 5: Meta (1) — main CLI skill