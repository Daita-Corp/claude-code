---
description: Debug production deployment issues with logs and diagnostics
---

You are helping the user debug issues with their deployed DAITA agents or workflows. Be systematic and thorough:

## Information Gathering

1. **Understand the problem**:
   - What symptoms are they seeing? (agent failing, wrong output, timeout, etc.)
   - When did it start? (recent deployment, been happening, intermittent)
   - Which agent/workflow is affected?

2. **Verify API access**:
   - Check if `DAITA_API_KEY` is set
   - If not, explain they need it to access production logs and STOP

3. **Check deployment status**:
   - Run `daita status` to see current deployment state
   - Run `daita deployments list --limit 5` to see recent deployments
   - Identify which deployment is currently active

## Log Analysis

4. **Fetch production logs**:
   - Run `daita logs production` to get recent execution logs
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
   - Run `daita test [agent-name]` locally to see if the issue reproduces
   - Compare local vs production behavior
   - This helps identify environment-specific issues

7. **Check configuration**:
   - Read `daita-project.yaml` to verify agent configuration
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
     - Adjust configuration in `daita-project.yaml`
   - Ask if user wants you to apply the fixes

10. **Apply fixes and redeploy**:
    - If user agrees, make the necessary code/config changes
    - Run `daita test` to verify fixes work locally
    - Use `/ship` command to redeploy safely

## Alternative Diagnostics

11. **If no obvious errors**:
    - Check for timeout issues (suggest increasing timeout in config)
    - Check for rate limiting (suggest backoff strategies)
    - Check input data format (suggest data validation)
    - Review LLM model configuration (suggest different model if needed)

12. **Rollback option**:
    - If the issue started with a recent deployment, suggest rollback
    - Show command: `daita deployments rollback <previous-deployment-id>`

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
