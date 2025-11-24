---
description: Debug production deployment issues with logs and diagnostics
---

## Daita Framework Context

You are working with the **Daita AI Agents Framework** for debugging production deployments.

### Agent Architecture

**SubstrateAgent** - Agents run autonomously using tools:
```python
from Daita import SubstrateAgent
from Daita.core.tools import tool

@tool
async def my_tool(param: str) -> dict:
    '''Tool description'''
    return {"result": param}

agent = SubstrateAgent(name="Agent", model="gpt-4o-mini", prompt="...")
agent.register_tool(my_tool)
```

**Execution Flow**:
1. User/webhook triggers agent
2. Agent analyzes request and decides which tools to call
3. Tools execute and return results
4. Agent synthesizes final response
5. All operations logged automatically

### Common Production Issues

**Missing API Keys**:
- Symptom: "API key not found" or authentication errors
- Cause: Missing keys in `.env` file
- Fix: Add OPENAI_API_KEY, ANTHROPIC_API_KEY, etc. to `.env`

**Import Errors**:
- Symptom: "ModuleNotFoundError" or "ImportError"
- Cause: Missing dependencies in production
- Fix: Add to `requirements.txt` and redeploy

**Tool Execution Failures**:
- Symptom: Tool raises exception during execution
- Cause: Invalid input, missing data, external API failures
- Fix: Add error handling to tools, validate inputs

**Timeout Issues**:
- Symptom: "Execution timed out after 120s"
- Cause: Tool takes too long, LLM response slow, infinite loop
- Fix: Optimize tool code, use streaming, increase timeout in config

**Data Format Issues**:
- Symptom: "Unexpected data format" or parsing errors
- Cause: Webhook payload doesn't match field mapping
- Fix: Update field_mapping in Daita-project.yaml

### CLI Commands for Debugging

**View logs**:
- `Daita logs production` - Recent production logs
- `Daita logs production --follow` - Stream logs in real-time
- `Daita logs production --agent [name]` - Filter by agent

**Check status**:
- `Daita status` - Current deployment status
- `Daita deployments list` - Recent deployments
- `Daita deployments list --limit 10` - More history

**Local testing**:
- `Daita test [agent-name]` - Reproduce issue locally
- `Daita test --verbose` - Detailed test output
- `Daita test --data test.json` - Test with specific data

**Deployment management**:
- `Daita deployments rollback <id>` - Rollback to previous version
- `Daita webhook list` - Check webhook configuration

### Project Configuration

**Daita-project.yaml**:
```yaml
name: my-project
version: 1.0.0

agents:
  - name: my_agent
    display_name: "My Agent"
    webhooks:
      - slug: "webhook-name"
        field_mapping:
          "payload.data": "agent_input"

workflows:
  - name: my_workflow
```

**Environment (.env)**:
```bash
# LLM API Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Cloud deployment
Daita_API_KEY=Daita-...
```

### Log Analysis Patterns

**Look for these in logs**:
- Error messages and stack traces
- Tool call sequences (which tools were called)
- Execution times (identify slow operations)
- Token usage (identify expensive operations)
- HTTP status codes (API failures)

**Example log entry**:
```json
{
  "timestamp": "2024-01-15T10:30:45Z",
  "agent": "fraud_detector",
  "status": "error",
  "error": "OpenAI API timeout",
  "execution_time_ms": 30000,
  "tool_calls": ["check_velocity", "analyze_patterns"]
}
```

### Debugging Workflow

1. **Fetch logs** → Identify error message
2. **Reproduce locally** → Run `Daita test [agent]`
3. **Diagnose cause** → Check config, code, dependencies
4. **Apply fix** → Update code/config
5. **Test locally** → Verify fix works
6. **Redeploy** → Use `/ship` command
7. **Verify** → Check logs again

### Production vs Local Differences

**Production environment**:
- AWS Lambda execution (timeout limits)
- Environment variables from deployment
- Network restrictions (VPC, security groups)
- Cold starts (first execution slower)

**Local environment**:
- No timeout limits
- Local .env file
- Full network access
- Faster execution

Issues that only happen in production are usually:
- Environment variable mismatches
- Lambda timeouts
- Network/firewall restrictions
- Cold start delays

### Need More Info?

If you're unsure about error patterns, agent behavior, or debugging techniques:
- Check project's `CLAUDE.md` for custom patterns
- Visit **https://docs.Daita-tech.io/troubleshooting** for guides
- Review agent code in `agents/` directory
- Check `Daita-project.yaml` for configuration

---

## Command Instructions

You are helping the user debug issues with their deployed Daita agents or workflows. Be systematic and thorough:

## Information Gathering

1. **Understand the problem**:
   - What symptoms are they seeing? (agent failing, wrong output, timeout, etc.)
   - When did it start? (recent deployment, been happening, intermittent)
   - Which agent/workflow is affected?

2. **Verify API access**:
   - Check if `Daita_API_KEY` is set
   - If not, explain they need it to access production logs and STOP

3. **Check deployment status**:
   - Run `Daita status` to see current deployment state
   - Run `Daita deployments list --limit 5` to see recent deployments
   - Identify which deployment is currently active

## Log Analysis

4. **Fetch production logs**:
   - Run `Daita logs production` to get recent execution logs
   - Look for error messages, stack traces, or anomalies
   - If logs are too verbose, ask user which timeframe or agent to focus on

5. **Analyze the errors**:
   - Identify the root cause from logs:
     - Missing API keys or credentials
     - Import errors or missing dependencies
     - Tool execution failures
     - Timeout issues
     - Invalid input data
   - Explain what the error means in plain language

## Diagnosis

6. **Local reproduction** (if possible):
   - Run `Daita test [agent-name]` locally to see if the issue reproduces
   - Compare local vs production behavior
   - This helps identify environment-specific issues

7. **Check configuration**:
   - Read `Daita-project.yaml` to verify agent configuration
   - Check if `.env` file has all required API keys
   - Verify webhook configurations if applicable

8. **Review recent changes**:
   - Look at recent file modifications in agents/ or workflows/
   - Compare with previous working deployment if possible

## Solutions

9. **Propose fixes**:
   - Based on the diagnosis, suggest specific fixes:
     - Add missing API keys to `.env`
     - Fix code errors in agent files
     - Update dependencies in `requirements.txt`
     - Adjust configuration in `Daita-project.yaml`
   - Ask if user wants you to apply the fixes

10. **Apply fixes and redeploy**:
    - If user agrees, make the necessary code/config changes
    - Run `Daita test` to verify fixes work locally
    - Use `/ship` command to redeploy safely

## Alternative Diagnostics

11. **If no obvious errors**:
    - Check for timeout issues (suggest increasing timeout in config)
    - Check for rate limiting (suggest backoff strategies)
    - Check input data format (suggest data validation)
    - Review LLM model configuration (suggest different model if needed)

12. **Rollback option**:
    - If the issue started with a recent deployment, suggest rollback
    - Show command: `Daita deployments rollback <previous-deployment-id>`

## Summary

13. **Provide clear summary**:
    - What was the issue?
    - What caused it?
    - What was fixed?
    - How to verify it's working?
    - How to prevent it in the future?

**Important**:
- Be methodical and don't jump to conclusions
- Show relevant log excerpts to user
- Explain technical issues in understandable terms
- Always verify fixes locally before redeploying
- Provide preventive advice for the future
- Follow Daita debugging best practices
