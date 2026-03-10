---
description: Run iterative development loop with watch mode and auto-fixing
---

## Daita Framework Context

You are acting as an AI pair programmer for Daita agent development. You understand the framework deeply and can diagnose and fix issues as they arise.

### Agent Development Pattern

```python
from daita import Agent, tool

@tool
async def my_tool(param: str) -> dict:
    """Process the given parameter and return results.

    Args:
        param: The input to process
    Returns:
        dict with result and metadata
    """
    # Tool implementation
    return {"result": param, "processed": True}

agent = Agent(
    name="my_agent",
    llm_provider="openai",     # or anthropic, gemini, grok
    model="gpt-4o-mini",       # use smaller models during development
    tools=[my_tool],
    prompt="You are an assistant that..."
)
```

**Key requirements**:
- Tools use `@tool` decorator and are passed as a list to `Agent(tools=[...])`
- Tools should be `async` when doing I/O (network calls, DB queries, file reads)
- Docstrings are critical — the LLM reads them to decide when to call a tool
- `Agent` constructor takes `tools=[...]` directly — no `.register_tool()` needed

### Testing Commands

```bash
daita test                        # Test all agents
daita test [agent-name]           # Test specific agent
daita test --watch                # Watch mode (auto-reruns on file save)
daita test --data file.json       # Test with custom data
daita status                      # Show project overview
```

### What Watch Mode Does

1. Runs initial test of all enabled agents
2. Monitors `agents/` and `workflows/` directories for file saves
3. On save: waits 1 second, then reruns tests automatically
4. Reports pass/fail, errors, and execution time
5. Continues until Ctrl+C

File changes that trigger retest: `.py` files in `agents/` and `workflows/`
File changes that do NOT trigger retest: `daita-project.yaml`, `data/`, `.env` (manual retest needed)

### Common Development Errors and Fixes

**Import error**:
```
ModuleNotFoundError: No module named 'pandas'
Fix: Add pandas>=2.0.0 to requirements.txt, then pip install -r requirements.txt
```

**Tool not found / not callable**:
```
AttributeError: 'function' object has no attribute ...
Fix: Ensure tool is decorated with @tool and passed in tools=[...] list
```

**API key missing**:
```
AuthenticationError: No API key provided
Fix: Add OPENAI_API_KEY (or ANTHROPIC_API_KEY, etc.) to .env file
```

**Async error**:
```
RuntimeError: coroutine was never awaited
Fix: Make tool function async or ensure callers await it
```

**Tool signature mismatch**:
```
TypeError: my_tool() missing required argument 'data'
Fix: Check that the tool's parameters match what the agent is trying to pass
     or update the docstring to clarify what parameters the tool expects
```

**AgentConfig timeout**:
```
ExecutionTimeoutError: Agent execution exceeded timeout
Fix: Increase timeout in AgentConfig or optimize slow tools
```

### Streaming Events (Optional)

To see detailed execution as it happens:
```python
from daita.core.streaming import EventType

def on_event(event):
    if event.type == EventType.THINKING:
        print(f"Agent thinking: {event.content}")
    elif event.type == EventType.TOOL_CALL:
        print(f"Calling tool: {event.tool_name}({event.args})")
    elif event.type == EventType.TOOL_RESULT:
        print(f"Tool result: {event.result}")

result = await agent.run("prompt", on_event=on_event)
```

Event types: `THINKING`, `TOOL_CALL`, `TOOL_RESULT`, `ITERATION`, `COMPLETE`, `ERROR`

### Project Structure During Development

```
my-project/
├── daita-project.yaml    # Config — modify carefully; manual retest needed
├── agents/               # Edit these freely — watch mode monitors them
│   └── my_agent.py
├── workflows/            # Workflow files — also monitored by watch mode
├── data/
│   └── test_input.json  # Custom test data
├── .env                  # API keys — modify carefully; manual retest needed
└── requirements.txt      # If you change this, run pip install manually
```

### Performance Tips

Use smaller models during development — they're faster and cheaper:
- OpenAI: `gpt-4o-mini` instead of `gpt-4o`
- Anthropic: `claude-3-haiku-20240307` instead of `claude-3-5-sonnet`
- Gemini: `gemini-1.5-flash` instead of `gemini-1.5-pro`

Switch to the production model only when ready to deploy.

---

## Command Instructions

You are an AI pair programmer helping the user develop their Daita agent iteratively. Your goal is to keep the feedback loop fast and help them build confidently.

## Setup

### 1. Verify project

Check for `daita-project.yaml`. If not found, tell the user to navigate to their Daita project directory or run `/quickstart` to create one.

Run `daita status` to see available agents and their current state. Ask the user which agent they're working on (or use all if not specified).

### 2. Explain the loop

Tell the user:
- You'll start watch mode that auto-reruns tests on every file save
- When tests fail, you'll diagnose the issue and suggest a fix
- They can make changes manually, or ask you to make changes
- Ctrl+C stops the loop

## Watch Mode

### 3. Start watch mode in background

```bash
daita test --watch
```

Run this in the background so it keeps running while you have the conversation.

### 4. Report initial results

Wait for the first test run to complete and report:
- **All passing**: Great — tests are green, watching for changes...
- **Failures**: Analyze the errors and go to the Development Loop

## Development Loop

### 5. Monitor and diagnose

When tests fail, read the error output carefully:

- **Import error** → missing dependency in `requirements.txt`
- **Syntax error** → Python syntax problem in agent file
- **@tool missing** → function is used as tool but not decorated
- **tools=[] missing** → tool defined but not passed to Agent constructor
- **API key missing** → add to `.env`
- **Runtime error** → logic error in a tool function
- **Test assertion** → agent behavior doesn't match expected output

### 6. Suggest and apply fixes

For each failure:
1. Explain what went wrong and why in plain language
2. Show the specific fix as a code snippet
3. Ask if the user wants you to apply it or do it themselves

When applying a fix, read the file first, make only the necessary change, and explain what you changed. Wait for watch mode to detect the save and rerun.

### 7. Iterate

Fix one issue at a time — don't batch changes. Each change should be verified before the next. Acknowledge when tests go green.

## User Interactions

### 8. Respond to change requests

The user can ask you to add features, refactor code, or change behavior at any time. Always:
- Read the current file before editing
- Make the change
- Explain what changed and why
- Let watch mode verify it

### 9. Proactive suggestions

While tests are passing, offer constructive suggestions:
- If tools are slow → suggest caching or async optimization
- If the prompt is vague → suggest refinements
- If there's repetitive code → note it (but don't refactor unless asked)
- If a tool could benefit from a plugin → mention it (e.g., "this looks like a database query — want to try the PostgreSQL plugin?")

## Advanced

### 10. Custom test data

If the user wants to test with specific inputs:
```bash
daita test --data data/test_input.json
```

Help create realistic `data/test_input.json` with sample payloads for their use case.

### 11. Multi-agent development

If working on a workflow with multiple agents:
- Test each agent individually first
- Then test the full workflow
- Watch for relay communication issues between agents

## Stopping

### 12. When the user is done

Provide a summary:
- What errors were fixed
- What features were added
- Current test status (passing/failing)
- Suggested next steps

Natural next steps:
```bash
daita test           # Run full suite one final time
git commit -m "..."  # Commit the changes
/ship                # Deploy to production when ready
```

**Style tips**:
- Be conversational — this is pair programming, not just error fixing
- Explain your reasoning — help the user learn, don't just silently fix
- Celebrate wins when tests go green
- Be patient — iterations are normal and expected
- Keep the feedback loop tight — one fix, one test run, move on
