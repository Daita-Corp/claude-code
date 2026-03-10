# CLAUDE.md — Daita Claude Code Commands Repo

This repository contains Claude Code slash commands and skills for the **Daita AI Agents Framework**.

## Repo Structure

```
.claude/
  commands/          # Slash commands — one .md file per command
  skills/            # Reusable skills — invoked from commands or standalone
CLAUDE.md            # This file
README.md            # User-facing installation and overview
```

## Commands (`.claude/commands/`)

| File | Slash command | Purpose |
|---|---|---|
| `quickstart.md` | `/quickstart` | Bootstrap a new Daita project |
| `ship.md` | `/ship` | Deploy to production with safety checks |
| `dev-loop.md` | `/dev-loop` | Iterative dev with watch mode |
| `setup-webhook.md` | `/setup-webhook` | Configure webhook integrations |
| `debug-deployment.md` | `/debug-deployment` | Diagnose production issues |
| `add-plugin.md` | `/add-plugin` | Add a data source plugin to an agent |
| `create-workflow.md` | `/create-workflow` | Build a multi-agent workflow |
| `add-provider.md` | `/add-provider` | Switch or add an LLM provider |
| `setup-memory.md` | `/setup-memory` | Add persistent memory to an agent |

## Skills (`.claude/skills/`)

| File | Purpose |
|---|---|
| `agent-scaffold.md` | Generate complete agent boilerplate |
| `tool-scaffold.md` | Generate a @tool function |
| `test-scaffold.md` | Generate a pytest test file for an agent |

## Daita Framework Quick Reference

This is the **correct** current API. Use these patterns in all commands and skills.

### Agent class (primary interface)

```python
from daita import Agent, tool

@tool
async def my_tool(param: str) -> dict:
    """What this tool does and when to call it."""
    return {"result": param}

agent = Agent(
    name="my_agent",
    llm_provider="openai",       # openai | anthropic | gemini | grok
    model="gpt-4o-mini",
    tools=[my_tool],
    prompt="You are a..."
)

result = await agent.run("prompt")
result = await agent.run_detailed("prompt")  # includes cost/tokens/time
```

### Providers and models

| Provider | `llm_provider` | Example models |
|---|---|---|
| OpenAI | `"openai"` | `"gpt-4o"`, `"gpt-4o-mini"` |
| Anthropic | `"anthropic"` | `"claude-3-5-sonnet-20241022"`, `"claude-3-haiku-20240307"` |
| Gemini | `"gemini"` | `"gemini-1.5-pro"`, `"gemini-1.5-flash"` |
| Grok | `"grok"` | `"grok-beta"` |

### CLI commands (all lowercase `daita`)

```bash
daita init [project-name]
daita create agent <name>
daita create workflow <name>
daita test [agent-name]
daita test --watch
daita status
daita push
daita push --dry-run
daita deployments list
daita deployments show <id>
daita webhooks list
daita logs
daita logs --follow
daita secrets
```

### Config file: `daita-project.yaml`

```yaml
name: my-project
version: 1.0.0
description: ...
created_at: '2026-01-01T00:00:00'

agents:
  - name: my_agent
    description: ...
    file: agents/my_agent.py
    enabled: true
    webhooks:
      - slug: "event-slug"
        field_mapping:
          "payload.field": "param_name"

workflows:
  - name: my_workflow
    file: workflows/my_workflow.py
    enabled: true
```

### Environment variables

```bash
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=...
GROK_API_KEY=...
DAITA_API_KEY=daita-...     # required for cloud deployment
```

### Plugins

```python
from daita.plugins import PostgreSQLPlugin, SlackPlugin, S3Plugin, PineconePlugin
# ... 27+ plugins available

agent.add_plugin(PostgreSQLPlugin(host=..., database=..., user=..., password=...))
```

### Workflows

```python
from daita.core.workflow import Workflow, Connection

workflow = Workflow(name="my-workflow")
workflow.add_agent(agent_a)
workflow.add_agent(agent_b)
workflow.add_connection(Connection(from_agent="agent_a", to_agent="agent_b"))
```

### Memory

```python
from daita.plugins.memory import MemoryPlugin

memory = MemoryPlugin(uri=os.environ["MEMORY_URI"])
agent.add_plugin(memory)
```

## Common Mistakes to Avoid

| Wrong (old API) | Correct (current API) |
|---|---|
| `from Daita import SubstrateAgent` | `from daita import Agent` |
| `agent.register_tool(my_tool)` | `Agent(tools=[my_tool])` |
| `def create_agent():` factory | Not needed — instantiate directly |
| `Daita test` (capital D) | `daita test` |
| `Daita-project.yaml` | `daita-project.yaml` |
| `Daita webhook list` | `daita webhooks list` |
| `Daita_API_KEY` | `DAITA_API_KEY` |

## When editing commands or skills

1. Keep the framework context section accurate and minimal — don't pad it
2. Reference patterns from this CLAUDE.md rather than duplicating them
3. Test any code examples mentally against the real API
4. Keep command instructions action-oriented — tell Claude what to DO, not just what to know
