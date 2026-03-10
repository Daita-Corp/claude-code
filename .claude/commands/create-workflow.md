---
description: Create a multi-agent workflow where agents pass data to each other via the Relay system
---

## Daita Framework Context

You are building a **multi-agent workflow** using the Daita AI Agents Framework. Workflows connect multiple agents so they can collaborate — passing messages between agents via the Relay system.

### When to Use Workflows

Use workflows when:
- You have a **pipeline**: output of one agent feeds into another
- You need **specialization**: different agents are experts at different parts of a task
- You want **parallel processing**: multiple agents working simultaneously
- You have **fan-out/fan-in**: one agent delegates to several, then one collects results

Don't use workflows for tasks a single agent with multiple tools can handle.

### Workflow Architecture

```python
from daita import Agent, tool
from daita.core.workflow import Workflow, Connection

# Define agents
researcher = Agent(
    name="researcher",
    llm_provider="openai",
    model="gpt-4o",
    tools=[search_web, fetch_page],
    prompt="You research topics thoroughly and return structured findings."
)

writer = Agent(
    name="writer",
    llm_provider="anthropic",
    model="claude-3-5-sonnet-20241022",
    tools=[format_report],
    prompt="You write clear, engaging summaries from research findings."
)

# Connect agents into a workflow
workflow = Workflow(name="research_pipeline")
workflow.add_agent(researcher)
workflow.add_agent(writer)
workflow.add_connection(
    Connection(
        from_agent="researcher",
        to_agent="writer"
    )
)
```

### Connection Types

**Sequential** (A → B): Output of A becomes input to B
```python
workflow.add_connection(Connection(from_agent="extractor", to_agent="analyzer"))
```

**Fan-out** (A → B, A → C): A sends to both B and C
```python
workflow.add_connection(Connection(from_agent="dispatcher", to_agent="agent_b"))
workflow.add_connection(Connection(from_agent="dispatcher", to_agent="agent_c"))
```

**Fan-in** (B → D, C → D): Both B and C feed into D
```python
workflow.add_connection(Connection(from_agent="agent_b", to_agent="aggregator"))
workflow.add_connection(Connection(from_agent="agent_c", to_agent="aggregator"))
```

### Relay Messaging

Agents communicate via the RelayManager. Messages have status tracking:
- `PENDING` — sent, not yet received
- `ACKNOWLEDGED` — received and being processed
- `FAILED` — processing failed
- `TIMEOUT` — recipient didn't respond in time

### Running a Workflow

```python
import asyncio

async def main():
    await workflow.start()
    result = await workflow.run("Research the latest AI developments and write a summary")
    await workflow.stop()
    print(result)

asyncio.run(main())
```

### Project Configuration for Workflows

Add workflows to `daita-project.yaml`:
```yaml
name: my-project
version: 1.0.0

agents:
  - name: researcher
    description: Researches topics and returns structured findings
    file: agents/researcher.py
    enabled: true

  - name: writer
    description: Writes summaries from research
    file: agents/writer.py
    enabled: true

workflows:
  - name: research_pipeline
    description: Research and write pipeline
    file: workflows/research_pipeline.py
    enabled: true
```

### Workflow File Structure

Each workflow gets its own file in `workflows/`:
```python
# workflows/research_pipeline.py
import os
from daita import Agent, tool
from daita.core.workflow import Workflow, Connection

# Import tools
from agents.researcher import search_web, fetch_page
from agents.writer import format_report

def create_workflow():
    """Factory function for Daita CLI"""
    researcher = Agent(
        name="researcher",
        llm_provider="openai",
        model="gpt-4o",
        tools=[search_web, fetch_page],
        prompt="You research topics thoroughly..."
    )

    writer = Agent(
        name="writer",
        llm_provider="anthropic",
        model="claude-3-5-sonnet-20241022",
        tools=[format_report],
        prompt="You write summaries from research findings..."
    )

    workflow = Workflow(name="research_pipeline")
    workflow.add_agent(researcher)
    workflow.add_agent(writer)
    workflow.add_connection(Connection(from_agent="researcher", to_agent="writer"))

    return workflow
```

### Mixed Provider Workflows

Different agents in the same workflow can use different LLM providers — this is a strength of Daita workflows:
```python
# Use each provider for what it's best at
extractor = Agent(name="extractor", llm_provider="openai", model="gpt-4o-mini", ...)     # fast, cheap
reasoner = Agent(name="reasoner", llm_provider="anthropic", model="claude-3-5-sonnet", ...) # deep reasoning
writer = Agent(name="writer", llm_provider="gemini", model="gemini-1.5-pro", ...)        # long context
```

### CLI Commands for Workflows

```bash
daita create workflow <name>      # Scaffold a workflow file
daita test <workflow-name>        # Test the workflow
daita status                      # Verify workflow appears in project
daita push                        # Deploy (agents + workflows together)
```

---

## Command Instructions

You are helping the user design and implement a multi-agent workflow. Take time to understand the problem before writing code — workflow architecture decisions are hard to undo.

## Step 1 — Understand the problem

Ask:
- What is the overall goal? What should the workflow accomplish?
- Can you break it into distinct stages or roles? What does each stage do?
- Does any stage need specialized capabilities (fast vs. smart, cheap vs. powerful)?
- Are any stages independent and could run in parallel?

Sketch the data flow before writing any code:
```
Input → [Agent A: extract data] → [Agent B: analyze] → [Agent C: report] → Output
```

## Step 2 — Design the agents

For each agent in the workflow:
- Choose a focused, single-responsibility role
- Select the right LLM provider and model for that role
- Design 1-3 tools that support the role
- Write a clear `prompt` that describes what the agent does and what it receives

Avoid the common mistake of making agents too general — each agent should be a specialist.

## Step 3 — Decide connection topology

Choose the connection pattern:
- **Pipeline** (A→B→C): each stage processes then passes to next
- **Fan-out** (A→B, A→C): A delegates to specialized agents
- **Fan-in** (B→D, C→D): multiple agents converge on a collector
- **Mixed**: combination of the above

## Step 4 — Scaffold the files

```bash
daita create workflow [workflow-name]
```

Then create or update agent files in `agents/` if they don't exist yet. Each agent that's part of the workflow needs its own file.

### Write the workflow file

Create `workflows/[workflow-name].py` with:
- All agent definitions
- Workflow + Connection setup
- `create_workflow()` factory function

### Write individual agent files

Each agent should also be usable standalone (for testing). Structure them as individual `agents/[name].py` files, then import tools into the workflow file.

## Step 5 — Update daita-project.yaml

Add all new agents and the workflow to `daita-project.yaml`. Make sure `enabled: true` for everything.

## Step 6 — Test each agent independently

```bash
daita test researcher
daita test writer
```

Fix individual agents before testing the full workflow.

## Step 7 — Test the full workflow

```bash
daita test research_pipeline
```

Watch for relay communication errors. If agents aren't passing data correctly, check:
- Connection `from_agent` and `to_agent` names match the agent `name` field
- The receiving agent's `prompt` tells it what kind of input to expect
- No timeout issues between agents

## Step 8 — Summary

Provide a clear summary of the workflow:
- A diagram of the data flow (text-based)
- What each agent does
- Which LLM provider/model each agent uses
- How to test it and deploy it

**Important**: Test agents individually before testing the full workflow — it's much easier to debug in isolation. Workflows are powerful but add complexity; if a single agent with more tools could solve the problem, that's simpler and better.
