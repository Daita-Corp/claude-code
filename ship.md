---
description: Deploy your Daita project to production with safety checks
---

## Daita Framework Context

You are working with the **Daita AI Agents Framework** - a production-ready framework for deploying autonomous AI agents to the cloud.

### Key Components

**SubstrateAgent** - Main agent class:
```python
from Daita import SubstrateAgent

agent = SubstrateAgent(
    name="Agent Name",
    model="gpt-4o-mini",
    prompt="System prompt"
)
```

**Tools** - Define with @tool decorator:
```python
from Daita.core.tools import tool

@tool
async def tool_name(param: str) -> dict:
    '''Tool description'''
    return {"result": param}
```

**APIs**:
- `run(prompt)` - Simple execution
- `run_detailed(prompt)` - With metadata (cost, time, tokens)

### CLI Commands for Deployment

**Pre-deployment**:
- `Daita test` - Test all agents locally
- `Daita test [agent-name]` - Test specific agent
- `Daita status` - Check project configuration

**Deployment**:
- `Daita push` - Deploy to cloud
- `Daita push --dry-run` - Preview deployment without changes
- `Daita push --verbose` - Detailed deployment output

**Post-deployment**:
- `Daita deployments list` - View deployment history
- `Daita webhook list` - List webhook URLs
- `Daita logs` - View execution logs
- `Daita status` - Check deployment status

### Project Configuration (Daita-project.yaml)

```yaml
name: my-project
version: 1.0.0              # IMPORTANT: Increment for each deployment
description: Project description

agents:
  - name: agent_file_name   # Matches agents/agent_file_name.py
    display_name: "Human Readable Name"
    type: substrate

  - name: another_agent
    display_name: "Another Agent"
    webhooks:                # Optional webhook configuration
      - slug: "webhook-name"
        field_mapping:
          "source.field": "target_field"

workflows:
  - name: workflow_name
    display_name: "Workflow Name"
```

### Version Management

Daita uses semantic versioning (MAJOR.MINOR.PATCH):
- **PATCH** (1.0.0 → 1.0.1): Bug fixes, small changes
- **MINOR** (1.0.0 → 1.1.0): New features, backward compatible
- **MAJOR** (1.0.0 → 2.0.0): Breaking changes

Always increment version before deployment for proper tracking.

### Environment Variables

Required for deployment (stored in `.env`):
```bash
# LLM API Keys (for agent execution)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Daita API Key (for cloud deployment)
Daita_API_KEY=Daita-...  # Get from https://Daita-tech.io
```

### Deployment Architecture

When you run `Daita push`:
1. Project is packaged (agents, workflows, config, .env)
2. Package uploaded to Daita cloud infrastructure
3. AWS Lambda functions created/updated for each agent
4. Webhooks configured (if specified)
5. Monitoring and logging enabled automatically

### Common Deployment Issues

- **Tests fail**: Fix locally first, never deploy failing code
- **Missing Daita_API_KEY**: Get from https://daita-tech.io
- **Missing LLM keys**: Add to `.env` file
- **Version conflicts**: Increment version in daita-project.yaml
- **Import errors**: Check requirements.txt has all dependencies

### Need More Info?

If you're unsure about deployment patterns or configuration:
- Check project's `CLAUDE.md` for custom patterns
- Visit **https://docs.daita-tech.io** for deployment guides
- Review `Daita-project.yaml` for current configuration

---

## Command Instructions

You are helping the user deploy their Daita project to production. This is a critical operation, so follow these safety checks:

## Pre-flight Checks

1. **Verify we're in a Daita project**:
   - Check for `Daita-project.yaml` in current directory or parent directories
   - If not found, tell user they need to be in a Daita project directory

2. **Run local tests first**:
   - Run `Daita test` to validate all agents and workflows work locally
   - If tests fail, STOP and help fix the issues before deploying
   - Show test results clearly

3. **Check project status**:
   - Run `Daita status` to review the project configuration
   - Check for any configuration issues or missing files
   - Verify that agents and workflows are properly configured

4. **Verify API key**:
   - Check if `Daita_API_KEY` is set in environment
   - If not set, explain how to get one from Daita-tech.io and STOP
   - Don't proceed without a valid API key

5. **Check version**:
   - Read the current version from `Daita-project.yaml`
   - Ask user if they want to increment the version (suggest patch, minor, or major based on changes)
   - If yes, update the version in `Daita-project.yaml`

## Deployment

6. **Deploy to production**:
   - Run `Daita -v push` to deploy
   - Monitor the output for any errors
   - If deployment fails, help diagnose and fix the issue

7. **Verify deployment**:
   - Run `Daita deployments list` to confirm the deployment succeeded
   - Show the deployment ID and timestamp

8. **Show webhook URLs** (if applicable):
   - Run `Daita webhook list` to display webhook URLs
   - Explain how to use them for integrations

## Post-deployment

9. **Provide summary**:
   - Summarize what was deployed (agents, workflows, version)
   - Show the deployment ID for reference
   - Provide commands to monitor the deployment:
     - `Daita logs production` - View execution logs
     - `Daita status` - Check deployment status
     - `Daita deployments list` - View deployment history

10. **Next steps**:
    - Suggest testing the deployed agents remotely
    - Explain how to view logs and monitor performance

**Important**:
- NEVER deploy if tests fail
- ALWAYS increment version for tracking
- Provide clear error messages and solutions
- Make the user feel confident about the deployment
- Follow Daita deployment best practices
