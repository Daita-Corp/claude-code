---
description: Set up webhook integration for external service triggers
---

You are helping the user set up webhook integration for their DAITA agent or workflow. Webhooks allow external services to trigger agent execution.

## Information Gathering

1. **Understand the integration**:
   - Which agent/workflow should the webhook trigger?
   - What external service will send webhooks? (GitHub, Slack, Stripe, custom service, etc.)
   - What event should trigger it? (e.g., "git push", "new Slack message", "payment received")

2. **Verify project setup**:
   - Check if we're in a DAITA project directory
   - Run `daita status` to see available agents/workflows
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

5. **Update daita-project.yaml**:
   - Read the current `daita-project.yaml` file
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
   - Check if `DAITA_API_KEY` is set
   - Run `daita push` to deploy with new webhook configuration
   - Monitor deployment for success

8. **Get webhook URL**:
   - Run `daita webhook list` to display all webhook URLs
   - Find and highlight the newly created webhook URL
   - Format: `https://api.daita-tech.io/api/v1/webhooks/trigger/{org_id}/{webhook_slug}`

## Testing

9. **Provide test instructions**:
   - Show how to test the webhook with curl:
   ```bash
   curl -X POST 'https://api.daita-tech.io/api/v1/webhooks/trigger/{org_id}/{webhook_slug}' \
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
    - Suggest running `daita logs production` after testing to see execution
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
- Webhooks require production deployment (DAITA_API_KEY needed)
- Test thoroughly before connecting to production services
- Use field mapping to extract only needed data
- Webhook URLs are organization-specific and secure
