---
description: Set up webhook integration for external service triggers
---

## Daita Framework Context

You are working with the **Daita AI Agents Framework** to set up webhook integrations for external service triggers.

### Webhook Architecture

**How Webhooks Work in Daita**:
1. External service (GitHub, Slack, Stripe, etc.) sends HTTP POST to webhook URL
2. Daita receives payload and applies field mapping
3. Mapped data is passed to specified agent/workflow
4. Agent executes autonomously using its tools
5. Result is logged and can be returned to external service

**Webhook URL Format**:
```
https://api.Daita-tech.io/api/v1/webhooks/trigger/{org_id}/{webhook_slug}
```

### Agent Architecture

**SubstrateAgent** - Agents that webhooks trigger:
```python
from Daita import SubstrateAgent
from Daita.core.tools import tool

@tool
async def process_webhook_data(data: dict) -> dict:
    '''Process incoming webhook data'''
    return {"status": "processed", "result": data}

def create_agent():
    agent = SubstrateAgent(
        name="Webhook Handler",
        model="gpt-4o-mini",
        prompt="You process webhook data and take appropriate actions"
    )
    agent.register_tool(process_webhook_data)
    return agent
```

When webhook is triggered, the agent receives mapped data and can:
- Call tools to process the data
- Make API calls to external services
- Store data in databases
- Send notifications
- Trigger other workflows

### Webhook Configuration

**In Daita-project.yaml**:
```yaml
name: my-project
version: 1.0.0

agents:
  - name: github_handler
    display_name: "GitHub Handler"
    webhooks:
      - slug: "github-push"           # URL-friendly identifier
        field_mapping:                # Transform webhook payload
          repository.name: repo_name
          pusher.name: user_name
          commits[0].message: commit_message

  - name: stripe_handler
    display_name: "Payment Handler"
    webhooks:
      - slug: "payment-received"
        field_mapping:
          data.object.amount: amount
          data.object.customer: customer_id
          data.object.receipt_email: email
```

### Field Mapping Syntax

**JSONPath notation** for nested fields:
- `simple_field` - Top-level field
- `nested.field` - Nested object access
- `array[0]` - First array element
- `array[0].field` - Field in first array element
- `data.object.amount` - Deeply nested field

**Example Transformation**:

Incoming payload:
```json
{
  "repository": {"name": "my-repo"},
  "pusher": {"name": "john"},
  "commits": [{"message": "Fix bug"}]
}
```

With field_mapping:
```yaml
repository.name: repo_name
pusher.name: user_name
commits[0].message: commit_message
```

Agent receives:
```json
{
  "repo_name": "my-repo",
  "user_name": "john",
  "commit_message": "Fix bug"
}
```

### CLI Commands for Webhooks

**Configure and deploy**:
- Edit `Daita-project.yaml` to add webhook configuration
- `Daita push [environment]` - Deploy with webhooks
- `Daita webhook list` - List all webhook URLs
- `Daita webhook list --api-key-only` - Filter to your API key

**Monitor webhooks**:
- `Daita logs production` - View webhook executions
- `Daita logs production --agent [name]` - Filter by agent
- `Daita status` - Check deployment status

### Common Webhook Services

**GitHub** - Repository events:
- Push events, pull requests, issues, releases
- Webhook in Settings → Webhooks → Add webhook
- Content type: `application/json`

**Slack** - Messages and commands:
- Slash commands, event subscriptions, interactive messages
- Configure in Slack App settings → Event Subscriptions
- Subscribe to bot events

**Stripe** - Payment events:
- Payment success, subscription changes, disputes
- Dashboard → Developers → Webhooks → Add endpoint
- Select events to listen for

**Twilio** - SMS and calls:
- Incoming messages, call status updates
- Phone Number settings → Messaging → Webhook URL

**Custom Services**:
- Any service that can send HTTP POST with JSON payload
- Configure service to POST to your webhook URL

### Environment Variables

Required for webhook deployment:
```bash
# LLM API Keys (for agent execution)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Daita API Key (for deployment)
Daita_API_KEY=Daita-...  # Get from https://Daita-tech.io
```

### Testing Webhooks

**Local Testing with curl**:
```bash
curl -X POST 'https://api.Daita-tech.io/api/v1/webhooks/trigger/{org_id}/{slug}' \
     -H 'Content-Type: application/json' \
     -d '{
       "test": "data",
       "foo": {"bar": "value"}
     }'
```

**View execution logs**:
```bash
Daita logs production
# Look for webhook execution entries
```

### Security

**Webhook URLs are secure**:
- Organization-scoped (only your org can use)
- No authentication token needed in URL
- HTTPS only
- Rate-limited per organization

**Best practices**:
- Use specific field mapping (don't pass entire payload)
- Validate data in agent tools
- Monitor logs for unexpected patterns
- Test with sample payloads before production use

### Need More Info?

If you're unsure about webhook configuration, field mapping, or service integration:
- Check project's `CLAUDE.md` for custom patterns
- Visit **https://docs.Daita-tech.io/webhooks** for webhook guides
- Review `Daita-project.yaml` for current configuration
- Check agent code in `agents/` directory

---

## Command Instructions

You are helping the user set up webhook integration for their Daita agent or workflow. Webhooks allow external services to trigger agent execution.

## Information Gathering

1. **Understand the integration**:
   - Which agent/workflow should the webhook trigger?
   - What external service will send webhooks? (GitHub, Slack, Stripe, custom service, etc.)
   - What event should trigger it? (e.g., "git push", "new Slack message", "payment received")

2. **Verify project setup**:
   - Check if we're in a Daita project directory
   - Run `Daita status` to see available agents/workflows
   - Confirm the target agent/workflow exists

## Webhook Configuration

3. **Create webhook slug**:
   - Suggest a descriptive webhook slug based on the use case
   - Example: "github-push", "slack-message", "stripe-payment"
   - Must be URL-friendly (lowercase, hyphens only)

4. **Determine field mapping**:
   - Based on the external service, suggest common field mappings
   - **GitHub push webhook** example:
     ```yaml
     field_mapping:
       repository.name: repo_name
       pusher.name: user_name
       commits[0].message: commit_message
     ```
   - **Slack message webhook** example:
     ```yaml
     field_mapping:
       event.text: message_text
       event.user: user_id
       event.channel: channel_id
     ```
   - **Stripe payment webhook** example:
     ```yaml
     field_mapping:
       data.object.amount: amount
       data.object.customer: customer_id
       data.object.id: payment_id
     ```
   - **Custom webhook**: Ask user what fields they want to extract from the payload

5. **Update Daita-project.yaml**:
   - Read the current `Daita-project.yaml` file
   - Add webhook configuration to the appropriate agent or workflow:
   ```yaml
   agents:
     - name: my_agent
       webhooks:
         - slug: "webhook-slug"
           field_mapping:
             "source.field": "target_field"
   ```
   - Or for workflows:
   ```yaml
   workflows:
     - name: my_workflow
       webhooks:
         - slug: "webhook-slug"
           field_mapping:
             "source.field": "target_field"
   ```

6. **Explain field mapping**:
   - Show user how the payload will be transformed
   - Give examples of source payload and resulting data that reaches the agent
   - Use JSONPath notation for nested fields and array access

## Deployment

7. **Deploy the webhook**:
   - Explain that webhooks only work in production
   - Check if `Daita_API_KEY` is set
   - Run `Daita push` to deploy with new webhook configuration
   - Monitor deployment for success

8. **Get webhook URL**:
   - Run `Daita webhook list` to display all webhook URLs
   - Find and highlight the newly created webhook URL
   - Format: `https://api.Daita-tech.io/api/v1/webhooks/trigger/{org_id}/{webhook_slug}`

## Testing

9. **Provide test instructions**:
   - Show how to test the webhook with curl:
   ```bash
   curl -X POST 'https://api.Daita-tech.io/api/v1/webhooks/trigger/{org_id}/{webhook_slug}' \
        -H 'Content-Type: application/json' \
        -d '{"sample": "payload"}'
   ```
   - Customize the sample payload based on their use case

10. **Integration instructions**:
    - **For GitHub**: Show how to add webhook in repository settings
    - **For Slack**: Show how to configure slash command or event subscription
    - **For Stripe**: Show how to add webhook endpoint in Stripe dashboard
    - **For custom**: Explain how their service should POST to the URL

## Verification

11. **Monitor webhook execution**:
    - Suggest running `Daita logs production` after testing to see execution
    - Show how to verify the webhook was triggered successfully
    - Help diagnose if webhook doesn't fire as expected

12. **Provide summary**:
    - Webhook URL (with copy-paste formatting)
    - What triggers it
    - How data is mapped
    - Where to configure it in the external service
    - How to monitor executions

## Common Webhook Configurations

Reference these for quick setup:

**GitHub Push**:
```yaml
slug: "github-push"
field_mapping:
  repository.name: repo_name
  pusher.name: pusher
  commits[0].message: last_commit
  ref: branch
```

**Slack Slash Command**:
```yaml
slug: "slack-command"
field_mapping:
  text: command_text
  user_id: user
  channel_id: channel
```

**Stripe Payment Success**:
```yaml
slug: "payment-success"
field_mapping:
  data.object.amount: amount
  data.object.customer: customer_id
  data.object.receipt_email: email
```

**Twilio SMS**:
```yaml
slug: "sms-received"
field_mapping:
  Body: message_text
  From: phone_number
  MessageSid: message_id
```

**Important**:
- Webhooks require production deployment (Daita_API_KEY needed)
- Test thoroughly before connecting to production services
- Use field mapping to extract only needed data
- Webhook URLs are organization-specific and secure
- Follow Daita webhook best practices
