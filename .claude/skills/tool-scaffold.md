---
description: Generate a well-structured @tool function for a Daita agent
---

Generate a high-quality `@tool` decorated function for a Daita agent. The tool's docstring is what the LLM reads to decide when and how to call it — this is the most important part.

## What to generate

### Requirements (ask if not provided)

- **Tool name** (snake_case verb phrase, e.g. `query_customer_by_email`)
- **What the tool does** — one sentence description
- **Parameters** — names, types, whether required or optional
- **Return data** — what shape of dict does it return?
- **I/O involved** — does it call a database, API, file system, etc.?

### Tool template

```python
from daita import tool
from typing import Optional


@tool
async def [tool_name](
    [required_param]: [type],
    [optional_param]: Optional[[type]] = None,
) -> dict:
    """[One-sentence description of what this tool does and when the agent should call it].

    Use this tool when [describe the triggering condition — be specific].
    [Optional: mention what it should NOT be used for if there's a similar tool nearby.]

    Args:
        [required_param]: [Description. Include valid values if constrained.]
        [optional_param]: [Description. Include default behavior if not provided.]

    Returns:
        dict with:
            - [key]: [what it contains]
            - [key]: [what it contains]
            - error (str, optional): error message if something went wrong
    """
    try:
        # Implementation
        result = ...

        return {
            "[key]": result,
            "success": True
        }
    except [SpecificException] as e:
        return {
            "success": False,
            "error": str(e)
        }
    except Exception as e:
        return {
            "success": False,
            "error": f"Unexpected error: {str(e)}"
        }
```

## Quality rules

### Docstring (most important)
The LLM reads the docstring to decide when and how to call this tool. It must:
- Start with a direct action phrase: "Query...", "Send...", "Calculate...", "Fetch..."
- Say **when** to use it (not just what it does)
- Describe every parameter clearly — the LLM uses these descriptions to fill in arguments
- Document the return dict structure so the LLM knows what it'll get back

**Good docstring**:
```python
"""Search customer records by email address and return matching accounts.

Use this tool when you need to look up a customer and you have their email.
For phone-based lookups, use search_customer_by_phone instead.

Args:
    email: The customer's email address (e.g., 'alice@example.com')
    include_inactive: If True, include deactivated accounts in results. Default: False

Returns:
    dict with:
        - customers: list of matching customer records
        - count: number of matches found
        - error: error message if the lookup failed
"""
```

**Bad docstring** (too vague):
```python
"""Search customers."""
```

### Function signature
- Use `async def` for any I/O operations (DB queries, API calls, file reads)
- Use `def` only for pure computation (math, string processing, data transformation)
- Type-annotate all parameters and return value
- Use `Optional[type] = None` for optional parameters

### Error handling
- Always wrap I/O in `try/except`
- Catch specific exceptions before catching `Exception`
- Return errors as `{"success": False, "error": "..."}` — don't raise exceptions from tools
  (the LLM can handle error dicts; exceptions cause the agent run to fail)

### Return value
- Always return a dict (never a string, int, or list directly)
- Include a `success: True/False` field when the operation can fail
- Keep values JSON-serializable (no datetime objects, no custom classes — use `.isoformat()` for dates)

## Example outputs

**Database query tool**:
```python
@tool
async def get_order_by_id(order_id: str) -> dict:
    """Retrieve a specific order and its line items by order ID.

    Use this when you need complete details about a particular order.
    For listing multiple orders, use list_orders_for_customer instead.

    Args:
        order_id: The unique order identifier (format: 'ORD-XXXXXXXX')

    Returns:
        dict with:
            - order: dict with order details (id, status, total, created_at)
            - line_items: list of items in the order
            - found: True if order exists, False if not found
            - error: error message if the lookup failed
    """
    try:
        order = await db.query_one("SELECT * FROM orders WHERE id = %s", [order_id])
        if not order:
            return {"found": False, "order": None, "line_items": []}

        items = await db.query("SELECT * FROM line_items WHERE order_id = %s", [order_id])
        return {
            "found": True,
            "order": {**order, "created_at": order["created_at"].isoformat()},
            "line_items": items
        }
    except Exception as e:
        return {"found": False, "error": str(e)}
```

**API call tool**:
```python
@tool
async def send_slack_message(channel: str, message: str) -> dict:
    """Send a message to a Slack channel.

    Use this to notify the team about important events or findings.
    The channel must be a valid Slack channel ID (e.g., 'C01234ABCDE') or name (e.g., '#alerts').

    Args:
        channel: Slack channel ID or name to send the message to
        message: The message text to send (supports Slack markdown formatting)

    Returns:
        dict with:
            - sent: True if message was delivered successfully
            - timestamp: Slack message timestamp if sent
            - error: error message if sending failed
    """
    try:
        response = await slack_client.chat_postMessage(channel=channel, text=message)
        return {"sent": True, "timestamp": response["ts"]}
    except SlackApiError as e:
        return {"sent": False, "error": e.response["error"]}
    except Exception as e:
        return {"sent": False, "error": str(e)}
```

**Pure computation tool**:
```python
@tool
def calculate_risk_score(transaction_amount: float, user_age_days: int, country_code: str) -> dict:
    """Calculate a fraud risk score for a transaction based on amount, account age, and country.

    Use this before approving or flagging a transaction. Higher scores indicate higher risk.
    Score ranges: 0-30 low risk, 31-70 medium risk, 71-100 high risk.

    Args:
        transaction_amount: Transaction value in USD
        user_age_days: How many days old the user account is
        country_code: ISO 3166-1 alpha-2 country code (e.g., 'US', 'GB')

    Returns:
        dict with:
            - score: risk score from 0 to 100
            - level: 'low', 'medium', or 'high'
            - factors: list of risk factors that contributed to the score
    """
    factors = []
    score = 0

    if transaction_amount > 10000:
        score += 30
        factors.append("high_amount")
    if user_age_days < 30:
        score += 25
        factors.append("new_account")
    if country_code in HIGH_RISK_COUNTRIES:
        score += 20
        factors.append("high_risk_country")

    level = "low" if score < 31 else ("medium" if score < 71 else "high")
    return {"score": min(score, 100), "level": level, "factors": factors}
```
