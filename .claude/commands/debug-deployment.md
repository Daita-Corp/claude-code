---
description: Diagnose and fix production deployment issues with logs and systematic debugging
---

## Daita Framework Context

You are debugging a production **Daita AI Agents Framework** deployment. Be systematic — fetch real data before drawing conclusions.

### CLI Debugging Commands

```bash
# Check current state
daita status                          # Project config and deployment status
daita deployments list                # Recent deployment history
daita deployments show <id>           # Details of a specific deployment

# Logs
daita logs                            # Recent production logs
daita logs --follow                   # Stream live logs
daita logs --lines 200                # More log history

# Local reproduction
daita test [agent-name]               # Try to reproduce locally
daita test --verbose                  # Verbose output
daita test --data test.json           # Test with specific input

# Rollback
daita deployments list                # Find a stable deployment ID
# (contact support or check docs for rollback command)

# Secrets
daita secrets                         # Check production secrets are configured
daita webhooks list                   # Check webhook URLs (if using webhooks)
```

### Agent Architecture (for understanding traces)

When an agent runs, it:
1. Receives input (from API call, webhook, or direct invocation)
2. Sends the prompt + tool definitions to the LLM
3. LLM decides which tools to call
4. Tools execute and return results
5. LLM synthesizes a final response
6. All steps are traced automatically

**Execution trace events**:
- `THINKING` — LLM is deciding what to do
- `TOOL_CALL` — LLM is calling a tool with arguments
- `TOOL_RESULT` — Tool returned a result
- `ITERATION` — Starting another reasoning iteration
- `COMPLETE` — Execution finished
- `ERROR` — Something failed

### Common Production Issues

**Missing or invalid API key**:
- Symptoms: `AuthenticationError`, `Invalid API key`, `401 Unauthorized`
- Cause: LLM provider key not set in production secrets
- Fix: `daita secrets` → add the missing key

**Missing DAITA_API_KEY**:
- Symptoms: `403 Forbidden`, auth error during deployment
- Fix: Verify key is valid at daita-tech.io

**Import / ModuleNotFoundError**:
- Symptoms: `ModuleNotFoundError: No module named 'X'`
- Cause: Dependency in code but not in `requirements.txt`
- Fix: Add to `requirements.txt`, redeploy

**Tool execution failure**:
- Symptoms: Tool raises exception, agent returns error response
- Cause: Invalid input, external API down, logic error
- Fix: Add error handling to tool, check external service

**Agent timeout**:
- Symptoms: `ExecutionTimeoutError`, execution cut off mid-run
- Cause: Tool takes too long, slow external API, infinite retry loop
- Fix: Optimize tool, add timeout to external calls, check retry config

**Webhook payload mismatch**:
- Symptoms: Agent gets `None` or wrong values, KeyError in tools
- Cause: Incoming payload structure doesn't match `field_mapping` in `daita-project.yaml`
- Fix: Update field mapping, check payload format with the external service

**Focus DSL error**:
- Symptoms: `FocusEvaluationError`, unexpected empty results in tools
- Cause: Focus DSL query applied to tool results that don't match expected schema
- Fix: Check `focus` parameter on tool definition, validate result structure

**Relay / workflow communication error**:
- Symptoms: Workflow stalls, `MessageTimeoutError`, agents not receiving messages
- Cause: Workflow connection misconfigured, agent crashed mid-workflow
- Fix: Check `Workflow` connection definitions, verify each agent works independently first

**Cold start issue** (Lambda):
- Symptoms: First execution very slow or times out, subsequent ones fine
- Cause: AWS Lambda cold start — normal but can be mitigated
- Fix: Increase Lambda timeout config, or warm-up strategy

### Environment Differences

| Aspect | Local (`daita test`) | Production |
|---|---|---|
| Timeout | None | Lambda limit (configurable) |
| API keys | `.env` file | `daita secrets` |
| Network | Full access | VPC-restricted (if configured) |
| Startup | Fast | Cold starts possible |
| Logs | Console output | `daita logs` |

Issues that only happen in production (not locally) are almost always:
1. Missing secrets (`daita secrets` shows what's configured)
2. Lambda timeout (increase or optimize)
3. Network restrictions (VPC, firewall)
4. Cold start (first request after idle period)

### Log Entry Format

```json
{
  "timestamp": "2026-03-10T14:32:45Z",
  "agent": "fraud_detector",
  "status": "error",
  "error": "AuthenticationError: OpenAI API key invalid",
  "execution_time_ms": 245,
  "tool_calls": ["check_velocity"],
  "tokens_used": 0
}
```

---

## Command Instructions

You are debugging a production Daita deployment. Be methodical — gather data before proposing solutions. Production issues can have multiple causes; don't guess.

## Information Gathering

### 1. Understand the symptoms

Ask:
- What are they seeing? (error message, wrong output, timeout, no response, etc.)
- When did it start? (after a specific deployment, intermittently, always)
- Which agent or workflow is affected?
- Is it every execution or only certain inputs?

If they described the issue in their message, use that context and skip questions.

### 2. Verify API access

Check if `DAITA_API_KEY` is set. Without it, you can't access production logs. If it's missing, stop and explain how to set it.

### 3. Check deployment status

```bash
daita status
daita deployments list
```

Identify which deployment is currently active and when it was deployed. Compare the deployment timestamp with when the issue started — this often pinpoints the culprit deployment.

## Log Analysis

### 4. Fetch production logs

```bash
daita logs --lines 100
```

Look for:
- Error messages and stack traces
- Which tools were called before the error
- Execution times (identify slow operations)
- Any pattern in when failures occur

If logs are sparse or the failure is intermittent, try `daita logs --follow` and ask the user to trigger a failing execution while you watch.

### 5. Identify the root cause

Match the log evidence to the known issue patterns above. Explain what the error means in plain language — don't just paste the error back at them.

## Diagnosis

### 6. Local reproduction

If the issue could be a code bug (not an environment/secrets issue):
```bash
daita test [agent-name] --verbose
```

Compare local vs production. If it fails locally too, fix is simpler. If it only fails in production, focus on environment differences (secrets, timeout, network).

### 7. Check configuration

Read `daita-project.yaml` — look for misconfigured agents, wrong field mappings, or disabled entries that should be enabled.

```bash
daita secrets
```

Verify all required API keys are present in production (not just `.env` locally).

### 8. Review recent changes

Look at recently modified files in `agents/` and `workflows/`. If the issue started after a specific deployment, focus on what changed in that deploy.

## Solutions

### 9. Propose fixes

Based on your diagnosis, suggest specific fixes:
- Missing secret → `daita secrets` to add it
- Code bug → show the exact change needed
- Import error → add to `requirements.txt`
- Config error → show corrected `daita-project.yaml`
- Timeout → suggest timeout increase or optimization

Ask if the user wants you to apply the fix.

### 10. Apply, test, redeploy

If fixing code or config:
1. Make the change
2. Run `daita test [agent-name]` to verify locally
3. Use `/ship` to redeploy safely (it includes all the pre-flight checks)

### 11. If no obvious errors in logs

Investigate less obvious causes:
- **Rate limiting**: Check if error messages mention rate limits — add retry/backoff logic
- **Data format**: Log the incoming data to see what the agent actually received
- **Model behavior**: The LLM might be calling the wrong tool or misreading a tool docstring
- **Cold start**: If only the first request fails, this is a Lambda cold start issue

### 12. Rollback option

If the issue started with a specific deployment and you can't quickly fix it:
```bash
daita deployments list
# Identify the last working deployment ID
```

Ask the user if they'd like to roll back while you work on a proper fix.

## Summary

End with a clear incident summary:
- **Issue**: what was happening
- **Root cause**: why it was happening
- **Fix**: what was changed
- **Verification**: how to confirm it's resolved
- **Prevention**: what to do to avoid this in the future

**Important**: Don't guess — always look at actual logs before proposing solutions. The error message is almost always there if you look carefully enough. Explain things in plain language — not everyone is a framework expert.
