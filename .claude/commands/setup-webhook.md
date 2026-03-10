---
description: Set up webhook integration so external services can trigger your Daita agents
---

## Daita Framework Context

You are configuring webhook integrations for the **Daita AI Agents Framework**. Webhooks let external services (GitHub, Slack, Stripe, Twilio, etc.) trigger your agents automatically.

### How Webhooks Work

1. External service sends HTTP POST to your webhook URL
2. Daita receives payload and applies your field mapping
3. Mapped fields are passed as the agent's input
4. Agent executes autonomously using its tools
5. Result is logged (and optionally returned to the caller)

**Webhook URL format**:
```
https://api.daita-tech.io/api/v1/webhooks/trigger/{org_id}/{webhook_slug}
```

### Webhook Configuration (daita-project.yaml)

Add webhooks under the `agents` or `workflows` section:

```yaml
name: my-project
version: 1.0.0

agents:
  - name: github_handler
    description: Handles GitHub push events
    file: agents/github_handler.py
    enabled: true
    webhooks:
      - slug: "github-push"           # URL-friendly identifier (lowercase, hyphens)
        field_mapping:                # Transform incoming payload to agent params
          repository.name: repo_name
          pusher.name: user_name
          commits[0].message: commit_message
          ref: branch

  - name: stripe_handler
    description: Processes Stripe payment events
    file: agents/stripe_handler.py
    enabled: true
    webhooks:
      - slug: "payment-received"
        field_mapping:
          data.object.amount: amount
          data.object.customer: customer_id
          data.object.receipt_email: email

workflows:
  - name: alert_pipeline
    file: workflows/alert_pipeline.py
    enabled: true
    webhooks:
      - slug: "alert-trigger"
        field_mapping:
          alert.severity: severity
          alert.message: message
```

### Field Mapping Reference

Field mapping transforms the incoming JSON payload into parameters passed to your agent. Use dot notation for nested fields and array indexing:

| Syntax | Meaning | Example |
|---|---|---|
| `field` | Top-level field | `"amount": amount` |
| `obj.field` | Nested field | `"data.object.id": payment_id` |
| `arr[0]` | First array element | `"commits[0]`: last_commit` |
| `arr[0].field` | Field in first element | `"commits[0].message": msg` |

**Example transformation**:

Incoming GitHub push payload:
```json
{
  "repository": {"name": "my-repo"},
  "pusher": {"name": "alice"},
  "commits": [{"message": "Fix login bug", "id": "abc123"}],
  "ref": "refs/heads/main"
}
```

With field mapping:
```yaml
repository.name: repo_name
pusher.name: user_name
commits[0].message: commit_message
```

Agent receives:
```json
{"repo_name": "my-repo", "user_name": "alice", "commit_message": "Fix login bug"}
```

### Agent Pattern for Webhooks

Agents receiving webhook data work the same as any other agent — the mapped fields become the natural language context passed to the agent:

```python
from daita import Agent, tool

@tool
async def notify_slack(channel: str, message: str) -> dict:
    """Send a notification to the specified Slack channel."""
    # ... slack API call ...
    return {"sent": True, "channel": channel}

agent = Agent(
    name="github_handler",
    llm_provider="openai",
    model="gpt-4o-mini",
    tools=[notify_slack],
    prompt="""You handle GitHub push events. When a push comes in, you:
    1. Analyze the commit message
    2. Decide if it's worth notifying the team
    3. Send a Slack notification if appropriate"""
)
```

### CLI Commands for Webhooks

```bash
daita status                      # Verify agent/workflow configuration
daita push                        # Deploy (webhooks only work in production)
daita webhooks list               # List all webhook URLs after deployment
daita webhooks list --verbose     # Show full URL and field mapping details
daita logs                        # Monitor webhook executions
daita logs --follow               # Stream live webhook triggers
```

### Common Service Configurations

**GitHub** (push, PR, issues):
```yaml
slug: "github-push"
field_mapping:
  repository.name: repo_name
  pusher.name: pusher
  commits[0].message: last_commit
  ref: branch
```
Configure at: Repository → Settings → Webhooks → Add webhook
Content type: `application/json`

**Slack** (slash commands, event API):
```yaml
slug: "slack-command"
field_mapping:
  text: command_text
  user_id: user
  channel_id: channel
  team_id: team
```
Configure at: Slack App → Slash Commands or Event Subscriptions → Request URL

**Stripe** (payments, subscriptions):
```yaml
slug: "payment-success"
field_mapping:
  data.object.amount: amount
  data.object.customer: customer_id
  data.object.receipt_email: email
  type: event_type
```
Configure at: Stripe Dashboard → Developers → Webhooks → Add endpoint

**Twilio** (SMS, voice):
```yaml
slug: "sms-received"
field_mapping:
  Body: message_text
  From: phone_number
  MessageSid: message_id
```
Configure at: Twilio Console → Phone Number → Messaging → Webhook

**PagerDuty** (incident alerts):
```yaml
slug: "incident-alert"
field_mapping:
  messages[0].event: event_type
  messages[0].incident.title: title
  messages[0].incident.urgency: urgency
```

---

## Command Instructions

You are setting up a webhook integration for a Daita agent or workflow. Webhooks require a production deployment, so walk the user through the full process.

## Information Gathering

### 1. Understand the integration

Ask:
- Which agent or workflow should the webhook trigger?
- What external service will send the webhook? (GitHub, Slack, Stripe, Twilio, custom, etc.)
- What event should trigger it? (push, payment received, message, alert, etc.)
- What data from the payload does the agent actually need?

If the user mentioned all this in their message, skip the questions and proceed.

### 2. Verify project

```bash
daita status
```

Confirm the target agent or workflow exists and is enabled. Read `daita-project.yaml` to see the current configuration.

## Configuration

### 3. Choose a webhook slug

Suggest a descriptive slug based on the use case. Rules:
- Lowercase, hyphens only (no underscores or spaces)
- Should describe the event, not the source: `payment-received` not `stripe-webhook`
- Keep it short: `github-push`, `sms-received`, `incident-alert`

### 4. Design the field mapping

Based on the service type, suggest appropriate field mapping using the reference configurations above. For custom services, ask the user to share a sample payload so you can design the mapping.

Show the before/after transformation clearly:
```
Incoming: { "repository": {"name": "my-repo"}, "pusher": {...} }
Mapped:   { "repo_name": "my-repo", "user_name": "alice" }
Agent sees: "repo_name: my-repo, user_name: alice"
```

### 5. Update daita-project.yaml

Read the current file, then add or update the webhook configuration for the target agent or workflow. Make sure the slug is unique across the project.

### 6. Update agent if needed

If the agent's `prompt` doesn't mention handling webhook data, suggest an update that tells the agent what kind of data it will receive and what to do with it.

## Deployment

### 7. Check DAITA_API_KEY

Webhooks only work after deploying to production. Verify the key is set. If not, tell the user to add it (get it from daita-tech.io).

### 8. Deploy

```bash
daita push
```

Monitor for success. If it fails, diagnose and fix before continuing.

### 9. Get the webhook URL

```bash
daita webhooks list
```

Find the new webhook URL and display it prominently in a copyable format:
```
Your webhook URL:
https://api.daita-tech.io/api/v1/webhooks/trigger/{org_id}/{slug}
```

## Testing

### 10. Test with curl

Generate a realistic test command based on their service:
```bash
curl -X POST 'https://api.daita-tech.io/api/v1/webhooks/trigger/{org_id}/{slug}' \
     -H 'Content-Type: application/json' \
     -d '{
       "sample": "payload matching their service format"
     }'
```

### 11. Integration instructions

Provide service-specific setup instructions:
- **GitHub**: Repository → Settings → Webhooks → Add webhook → paste URL
- **Slack**: App settings → Event Subscriptions or Slash Commands → Request URL
- **Stripe**: Dashboard → Developers → Webhooks → Add endpoint → select events
- **Twilio**: Console → Phone Number → Messaging → Webhook URL
- **Custom**: Instruct them to POST JSON to the URL

### 12. Monitor

After testing:
```bash
daita logs --follow
```

Look for an entry showing the webhook triggered and the agent executed. Help diagnose if it doesn't appear.

## Summary

End with a clean summary:
- **Webhook URL** (prominent, copyable)
- **Triggers on**: description of the event
- **Data passed to agent**: the mapped fields
- **Where to configure**: service-specific location
- **How to monitor**: `daita logs --follow`

**Important**: Webhooks require cloud deployment — they don't work locally. Always test with curl before connecting the real external service.
