---
description: Bootstrap a new Daita project from scratch with intelligent setup
---

## Daita Framework Context

You are working with the **Daita AI Agents Framework** — a production-ready framework for building and deploying autonomous AI agents.

### Core Agent Pattern

**Agent** — the primary class for building agents:
```python
from daita import Agent, tool

@tool
async def my_tool(param: str) -> dict:
    """Analyze input and return results. Args: param: the input string to analyze."""
    return {"result": param, "length": len(param)}

agent = Agent(
    name="my_agent",
    llm_provider="openai",      # openai | anthropic | gemini | grok
    model="gpt-4o-mini",        # any valid model for the provider
    tools=[my_tool],
    prompt="You are a helpful assistant that analyzes data using the tools provided."
)
```

**Running an agent**:
```python
import asyncio

async def main():
    result = await agent.run("Analyze this: hello world")
    print(result)

asyncio.run(main())
```

**Detailed result with metadata**:
```python
result = await agent.run_detailed("Analyze this")
# Returns: {"result": "...", "cost": 0.002, "processing_time_ms": 847, "tokens_used": 312, ...}
```

### LLM Provider Options

| Provider | llm_provider value | Example models |
|---|---|---|
| OpenAI | `"openai"` | `"gpt-4o"`, `"gpt-4o-mini"` |
| Anthropic | `"anthropic"` | `"claude-3-5-sonnet-20241022"`, `"claude-3-haiku-20240307"` |
| Google Gemini | `"gemini"` | `"gemini-1.5-pro"`, `"gemini-1.5-flash"` |
| xAI Grok | `"grok"` | `"grok-beta"` |

Required API key env vars: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `GROK_API_KEY`

### Project Structure

```
my-project/
├── daita-project.yaml    # Project configuration (lowercase)
├── agents/               # Agent Python files
│   └── my_agent.py
├── workflows/            # Multi-agent workflow files (optional)
├── data/                 # Test data
├── .env                  # API keys (never commit this)
└── requirements.txt      # Python dependencies
```

### Project Configuration (daita-project.yaml)

```yaml
name: my-project
version: 1.0.0
description: Project description
created_at: '2026-01-01T00:00:00'

agents:
  - name: my_agent          # must match agents/my_agent.py filename
    description: Agent description
    file: agents/my_agent.py
    enabled: true

workflows: []
```

### CLI Commands

```bash
# Local development (no API key needed)
daita init [project-name]         # Create new project
daita create agent <name>         # Scaffold a new agent file
daita create workflow <name>      # Scaffold a new workflow
daita test [agent-name]           # Test agents locally
daita test --watch                # Watch mode — auto-reruns on file save
daita status                      # Show project config and status

# Cloud deployment (requires DAITA_API_KEY)
daita push                        # Deploy to production
daita push --dry-run              # Preview without deploying
daita deployments list            # View deployment history
daita webhooks list               # List webhook URLs
daita logs                        # View execution logs
daita logs --follow               # Stream logs in real-time
daita secrets                     # Manage production secrets
```

### Adding Plugins (Data Sources)

Daita has 27+ plugins for databases, vector DBs, cloud services, and APIs:
```python
from daita.plugins import PostgreSQLPlugin, SlackPlugin

pg = PostgreSQLPlugin(host="localhost", database="mydb", user="user", password="pass")
slack = SlackPlugin(token=os.environ["SLACK_TOKEN"])

agent = Agent(name="analyst", llm_provider="openai", model="gpt-4o", tools=[my_tool])
agent.add_plugin(pg)
agent.add_plugin(slack)
```

### Environment Variables

```bash
# .env file
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=...
GROK_API_KEY=...
DAITA_API_KEY=daita-...     # Required for cloud deployment
```

---

## Command Instructions

You are helping the user bootstrap a new Daita AI agent project. Be conversational — explain what you're doing at each step and tailor everything to their specific use case.

### Step 1 — Understand the use case

Ask the user what kind of agent they want to build. If they already described it in their message, use that. Good examples:
- "data analyst that queries PostgreSQL and summarizes results"
- "Slack bot that monitors alerts and pages on-call"
- "fraud detection agent that checks payment patterns"

Get specific — knowing the use case lets you generate better tools and test data.

### Step 2 — Choose LLM provider

Ask which LLM provider they want to use, or recommend one based on their use case:
- **OpenAI (gpt-4o-mini)** — fastest to start, great default
- **Anthropic (claude-3-5-sonnet)** — best reasoning, great for analysis tasks
- **Gemini (gemini-1.5-flash)** — good for high-throughput, multimodal tasks
- **Grok (grok-beta)** — good for real-time data tasks

Check if the relevant API key is already set in their environment. If not, note what they'll need to add to `.env`.

### Step 3 — Project initialization

Ask for a project name, or suggest one based on their use case (kebab-case, e.g. `fraud-detector`).

```bash
daita init [project-name]
cd [project-name]
```

If they're adding an agent to an existing project, skip init and go straight to Step 4.

### Step 4 — Create the agent

Run `daita create agent [agent-name]` to scaffold the file, then read it and rewrite it with:
- Relevant `@tool` functions for the use case (2-4 tools is ideal)
- A focused `prompt` that describes the agent's role and when to use each tool
- The right `llm_provider` and `model` they chose
- Realistic tool implementations (not just stubs — make them actually work)

Each tool must have:
- A clear docstring the LLM will use to decide when to call it
- Type annotations on all parameters and return value
- Async implementation when doing I/O

### Step 5 — Set up configuration and dependencies

- Check `.env` for the required API key. If missing, create/update the file with a placeholder and clear instructions.
- If you added any imports (pandas, httpx, etc.), add them to `requirements.txt`.
- Verify `daita-project.yaml` references the new agent file correctly.

### Step 6 — Create test data

Create `data/test_input.json` with 2-3 realistic test cases for their use case. Make the data specific enough to actually test the agent's tools.

### Step 7 — Run and validate

```bash
daita test [agent-name]
```

If there are errors, diagnose and fix them before moving on. Common issues:
- Missing API key → add to `.env`
- Import error → add to `requirements.txt`
- Tool error → check function signature and logic

### Step 8 — Show status and next steps

```bash
daita status
```

Summarize what was created, how to continue developing locally, and how to deploy when ready (`DAITA_API_KEY` needed for `daita push`).

**Important**: Make generated code production-quality, not toy examples. The agent should actually do something useful for the user's stated use case. Show excitement about what they're building — this is the fun part!
