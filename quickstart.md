---
description: Bootstrap a new Daita project from scratch with intelligent setup
---

## Daita Framework Context

You are working with the **Daita AI Agents Framework** - a production-ready framework for building autonomous AI agents with tool-based execution.

### Core Components

**SubstrateAgent** - Main agent class with autonomous LLM-driven execution:
```python
from Daita import SubstrateAgent

agent = SubstrateAgent(
    name="Agent Name",
    model="gpt-4o-mini",  # or gpt-4, claude-3-5-sonnet, etc.
    prompt="You are a helpful assistant that..."
)
```

**@tool decorator** - Define tools that agents can autonomously call:
```python
from Daita.core.tools import tool

@tool
async def analyze_data(data: list) -> dict:
    '''Analyze data and return insights.'''
    return {
        "count": len(data),
        "summary": "analysis results"
    }
```

**Execution APIs**:
- `run(prompt)` - Simple execution, returns string answer
- `run_detailed(prompt)` - Full execution with metadata (cost, tokens, time)

```python
await agent.start()

# Simple usage
answer = await agent.run("Analyze this data: [1,2,3]")

# Detailed usage with metadata
result = await agent.run_detailed("Analyze this data: [1,2,3]")
# result = {"result": "...", "cost": 0.002, "processing_time_ms": 847, ...}

await agent.stop()
```

### Agent Creation Pattern

Every agent file should have a `create_agent()` function:
```python
from Daita import SubstrateAgent
from Daita.core.tools import tool

@tool
async def my_tool(param: str) -> dict:
    '''Tool description for the LLM'''
    # Tool logic here
    return {"result": param}

def create_agent():
    '''Factory function required by Daita CLI'''
    agent = SubstrateAgent(
        name="My Agent",
        model="gpt-4o-mini",
        prompt="System prompt here"
    )
    agent.register_tool(my_tool)
    return agent

if __name__ == "__main__":
    import asyncio
    async def main():
        agent = create_agent()
        await agent.start()
        result = await agent.run("test prompt")
        print(result)
        await agent.stop()
    asyncio.run(main())
```

### Project Structure

```
my-project/
├── Daita-project.yaml    # Project configuration
├── agents/               # Agent Python files
│   └── my_agent.py      # Each agent file
├── workflows/            # Workflow files (optional)
├── data/                 # Test data
├── .env                  # API keys (OPENAI_API_KEY, etc.)
└── requirements.txt      # Dependencies
```

### CLI Commands Reference

**Local Development (Free)**:
- `Daita init [project-name]` - Create new project
- `Daita create agent <name>` - Generate agent file
- `Daita test [target]` - Test agents locally
- `Daita test --watch` - Watch mode for development
- `Daita status` - Show project status

**Cloud Deployment (Requires Daita_API_KEY)**:
- `Daita push [environment]` - Deploy to production
- `Daita logs [environment]` - View execution logs
- `Daita webhook list` - List webhook URLs
- `Daita deployments list` - View deployment history

### Configuration File (Daita-project.yaml)

```yaml
name: my-project
version: 1.0.0
description: Project description

agents:
  - name: my_agent              # matches agents/my_agent.py
    display_name: "My Agent"    # human-readable name
    type: substrate

workflows:
  - name: my_workflow
    display_name: "My Workflow"
```

### Environment Variables

Store in `.env` file at project root:
```bash
# LLM API Keys (choose one or more)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=...
GROK_API_KEY=...

# For cloud deployment
Daita_API_KEY=Daita-...
```

### Key Differences from Other Frameworks

- **Not like LangChain**: No chains - agents are autonomous with tools
- **Not like CrewAI**: Single agent focus (not multi-agent by default)
- **Tool-based execution**: Everything happens through tools, not prompt engineering alone
- **Production-first**: Built for deployment with observability and monitoring

### Need More Info?

If you're unsure about any Daita patterns, APIs, or configuration:
- Check the project's `CLAUDE.md` file if it exists (project-specific patterns)
- Visit **https://docs.Daita-tech.io** for complete documentation
- Read existing agent files in the `agents/` directory to see patterns in use

---

## Command Instructions

You are helping the user bootstrap a new Daita AI agent project. Follow these steps:

1. **Understand the use case**: Ask the user what kind of agent they want to build (e.g., "data analyzer", "customer support bot", "document processor"). If they already mentioned it in their message, use that.

2. **Project initialization**:
   - Ask for a project name (or suggest one based on the use case)
   - Run `Daita init [project-name]` to create the project structure
   - Navigate into the new project directory

3. **Create a custom agent**:
   - Run `Daita create agent [agent-name]` with a relevant name
   - Read the generated agent file
   - Enhance it based on the use case by:
     - Adding relevant tools using the `@tool` decorator
     - Updating the prompt to match the use case
     - Adding any necessary imports or plugins

4. **Set up configuration**:
   - Check if LLM API keys are set (look for OPENAI_API_KEY, ANTHROPIC_API_KEY, etc.)
   - If not set, create a `.env` file with placeholder keys and instructions
   - Update `requirements.txt` if you added any additional dependencies

5. **Create test data**:
   - Create a `data/test_input.json` file with sample data relevant to the use case
   - Make it realistic and useful for testing

6. **Run first test**:
   - Run `Daita test [agent-name]` to validate the setup
   - If there are errors, diagnose and fix them
   - Explain what the test did and what the results mean

7. **Show project status**:
   - Run `Daita status` to show the project overview
   - Explain next steps for development

**Important guidelines**:
- Be conversational and explain what you're doing at each step
- If anything fails, diagnose and fix it before moving on
- Make the generated code production-quality, not just examples
- Tailor everything to the user's specific use case
- Show excitement about what they're building
- Follow Daita patterns shown above (create_agent() function, @tool decorator, etc.)

At the end, provide a summary of:
- What was created
- How to continue development locally (free)
- How to deploy to production when ready (with Daita_API_KEY)
