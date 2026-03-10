---
description: Generate a complete, production-quality Daita agent file from a description
---

Generate a complete Daita agent file following all framework conventions. Use the information provided (or ask for it) to produce a well-structured, immediately runnable agent.

## What to generate

### 1. Gather requirements (if not already provided)

You need:
- **Agent name** (snake_case, e.g. `fraud_detector`)
- **Agent purpose** — what problem does it solve?
- **LLM provider** — openai / anthropic / gemini / grok (default: openai)
- **Model** — which model (default: gpt-4o-mini)
- **Tools** — what actions should the agent be able to take? (2-4 is ideal)
- **Plugins needed** — any database, API, or service connections?

If the user hasn't provided these, ask before generating.

### 2. Generate the agent file

Produce a complete `agents/[agent-name].py` file with this structure:

```python
"""
[Agent Name] — [One-line description of what this agent does]
"""
import asyncio
import os
from typing import Optional
from daita import Agent, tool
# Add plugin imports if needed, e.g.:
# from daita.plugins import PostgreSQLPlugin


# ── Tools ──────────────────────────────────────────────────────────────────

@tool
async def [tool_name]([param]: [type]) -> dict:
    """[Clear description of what this tool does and when the agent should call it].

    Args:
        [param]: [description]
    Returns:
        dict with [result description]
    """
    # Implementation
    return {"result": ..., "status": "success"}


# ── Agent ──────────────────────────────────────────────────────────────────

agent = Agent(
    name="[agent-name]",
    llm_provider="[provider]",
    model="[model]",
    tools=[tool1, tool2, ...],
    prompt="""[Clear, specific system prompt that:
    1. Describes the agent's role
    2. Tells it when to use each tool
    3. Specifies the output format
    4. Sets any constraints or rules]"""
)

# Add plugins if needed:
# pg = PostgreSQLPlugin(host=os.environ["DB_HOST"], ...)
# agent.add_plugin(pg)


# ── Run ────────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    async def main():
        result = await agent.run("[realistic test prompt for this agent's purpose]")
        print(result)

    asyncio.run(main())
```

### 3. Quality requirements for generated code

**Tools must**:
- Have `@tool` decorator
- Be `async` functions (use `async def`)
- Have complete docstrings explaining purpose and params
- Have type annotations on all parameters and return type (`-> dict`)
- Include basic error handling (`try/except`) for any I/O operations
- Return a dict (not a string or primitive)

**The system prompt must**:
- Clearly state the agent's role in 1-2 sentences
- List the available tools and when to use each one
- Specify the output format (e.g., "Return a JSON summary with fields: ...")
- Include any domain-specific rules (e.g., "Flag transactions over $10,000")

**The agent constructor must**:
- Use the exact tool function names in `tools=[...]`
- Use valid `llm_provider` and `model` values
- Include all required parameters

**Environment variables**:
- All credentials via `os.environ["KEY_NAME"]`
- Use `os.environ.get("KEY", "default")` only for optional config

### 4. Also generate the daita-project.yaml entry

Show what to add to `daita-project.yaml`:
```yaml
agents:
  - name: [agent-name]
    description: [one-line description]
    file: agents/[agent-name].py
    enabled: true
```

### 5. Show what environment variables are needed

List all required env vars with descriptions:
```bash
# .env additions needed:
OPENAI_API_KEY=sk-...         # LLM provider key
DB_HOST=localhost              # Database host (if applicable)
```

## Style guidelines

- Prefer `async def` for all tools
- Use descriptive variable names — no single-letter vars
- Keep tools focused — one clear action per tool
- The prompt should read like instructions to a smart analyst, not a robot
- Include realistic test prompts in the `if __name__ == "__main__":` block
- Don't add comments explaining obvious code — only add comments where logic isn't obvious
