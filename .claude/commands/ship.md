---
description: Deploy your Daita project to production with safety checks
---

## Daita Framework Context

You are working with the **Daita AI Agents Framework** to deploy agents to production. This is a safety-critical workflow — never deploy failing code.

### CLI Commands for Deployment

```bash
# Pre-deployment checks
daita test                        # Test all agents locally
daita test [agent-name]           # Test a specific agent
daita status                      # Check project configuration

# Deployment
daita push                        # Deploy to production
daita push --dry-run              # Preview what would be deployed
daita push --force                # Force deploy (skip confirmations)

# Post-deployment verification
daita deployments list            # View deployment history
daita deployments show <id>       # Show deployment details
daita webhooks list               # List webhook URLs
daita logs                        # View execution logs
daita logs --follow               # Stream logs in real-time
daita logs --lines 100            # Show last 100 log lines
daita secrets                     # Verify production secrets
```

### Project Configuration (daita-project.yaml)

```yaml
name: my-project
version: 1.0.1              # Increment before every deployment
description: Project description
created_at: '2026-01-01T00:00:00'

agents:
  - name: my_agent
    description: Agent description
    file: agents/my_agent.py
    enabled: true
    webhooks:                         # Optional
      - slug: "event-name"
        field_mapping:
          "payload.field": "param_name"

workflows: []
```

### Semantic Versioning

- **PATCH** (1.0.0 → 1.0.1): Bug fixes, small changes
- **MINOR** (1.0.0 → 1.1.0): New features, backward compatible
- **MAJOR** (1.0.0 → 2.0.0): Breaking changes to agent interface

Always increment version before deploying for proper tracking and rollback capability.

### Required Environment Variables

```bash
# .env file
OPENAI_API_KEY=sk-...           # or ANTHROPIC_API_KEY, GEMINI_API_KEY, etc.
DAITA_API_KEY=daita-...         # Required for cloud deployment
```

### Common Deployment Issues

| Issue | Symptom | Fix |
|---|---|---|
| Tests fail | Errors before deploy | Fix locally first — never deploy broken code |
| Missing DAITA_API_KEY | Auth error | Get key from daita-tech.io |
| Missing LLM key | Agent won't run | Add to `.env` and manage with `daita secrets` |
| Import error | ModuleNotFoundError | Add dependency to `requirements.txt` |
| Version conflict | Deploy rejected | Increment version in `daita-project.yaml` |

---

## Command Instructions

You are deploying a Daita project to production. This is a high-stakes operation — be methodical and never skip the safety checks.

## Pre-flight Checks

### 1. Verify we're in a Daita project

Check for `daita-project.yaml` in the current or parent directories. If not found, stop and tell the user they need to be in a Daita project directory (run `daita init` to create one).

### 2. Run local tests

```bash
daita test
```

If any tests fail, **stop here**. Help diagnose and fix the failures before proceeding. Show the test output clearly. Never deploy failing code — it will fail in production too.

### 3. Check project status

```bash
daita status
```

Review the output for configuration issues, missing files, or disabled agents. Flag anything that looks wrong.

### 4. Verify DAITA_API_KEY

Check if `DAITA_API_KEY` is set in the environment or `.env` file. If not set, stop and tell the user to:
1. Get their API key from daita-tech.io
2. Add it: `export DAITA_API_KEY=daita-...` or add to `.env`

### 5. Check and increment version

Read the current version from `daita-project.yaml`. Based on what's being deployed:
- Small fixes or tweaks → suggest PATCH
- New tools, agents, or features → suggest MINOR
- Changes to agent names or webhook slugs → suggest MAJOR

Ask the user which increment to use, then update `daita-project.yaml`.

### 6. Check production secrets

```bash
daita secrets
```

Verify that all API keys and secrets the agents need are configured for production, not just in the local `.env` file.

## Deployment

### 7. Deploy

```bash
daita push
```

Monitor the output carefully. If the deployment fails, read the error and help diagnose it. Common failures:
- **Import errors**: Package missing from `requirements.txt`
- **Auth errors**: Invalid `DAITA_API_KEY`
- **Config errors**: Malformed `daita-project.yaml`

### 8. Verify deployment succeeded

```bash
daita deployments list
```

Show the new deployment entry with its ID and timestamp. Confirm it shows as the active deployment.

### 9. Show webhook URLs (if applicable)

```bash
daita webhooks list
```

If the project has webhooks configured, display the URLs clearly in a copyable format. Remind the user to update any external services (GitHub, Slack, Stripe, etc.) if webhook URLs changed.

## Post-deployment

### 10. Summary

Provide a clear deployment summary:
- What was deployed (agents, workflows, version)
- Deployment ID for reference
- Webhook URLs (if any)

Monitoring commands to share:
```bash
daita logs --follow           # Watch live executions
daita logs --lines 50         # Recent history
daita deployments list        # Deployment history
daita status                  # Current state
```

**Important rules**:
- NEVER deploy if tests fail — fix first, then deploy
- ALWAYS increment the version — it enables rollback and tracking
- If the user seems rushed, gently insist on the pre-flight checks — production issues are much worse than a 60-second delay
