---
description: Run iterative development loop with watch mode and auto-fixing
---

## Daita Framework Context

You are working with the **Daita AI Agents Framework** in iterative development mode - acting as an AI pair programmer.

### Agent Development Patterns

**SubstrateAgent** - The agent class you'll be developing:
```python
from Daita import SubstrateAgent
from Daita.core.tools import tool

@tool
async def my_tool(param: str) -> dict:
    '''Tool description for the LLM'''
    # Tool implementation
    return {"result": param}

def create_agent():
    '''Required factory function for Daita CLI'''
    agent = SubstrateAgent(
        name="Agent Name",
        model="gpt-4o-mini",
        prompt="System prompt describing agent's role"
    )
    agent.register_tool(my_tool)
    return agent
```

**Key Requirements**:
- Every agent file must have a `create_agent()` function
- Tools must use `@tool` decorator
- Tools must be async functions
- Tools must have docstrings (LLM uses them)
- Agent must be started/stopped properly

### Testing Framework

**CLI Testing Commands**:
- `Daita test` - Test all agents and workflows
- `Daita test [agent-name]` - Test specific agent
- `Daita test --watch` - Watch mode (auto-rerun on changes)
- `Daita test --data file.json` - Test with custom data
- `Daita test --verbose` - Detailed test output

**Test Execution Flow**:
1. Daita CLI imports agent file
2. Calls `create_agent()` to get agent instance
3. Calls `agent.start()` to initialize
4. Calls `agent.run_detailed(test_prompt)` with test data
5. Validates response format and execution
6. Calls `agent.stop()` to cleanup
7. Reports results (pass/fail, time, cost)

**What Tests Check**:
- Agent file has `create_agent()` function
- Agent can be instantiated without errors
- Agent can start/stop properly
- Agent can execute `run()` and `run_detailed()` APIs
- Tools are registered correctly
- No import errors or missing dependencies

### Common Development Errors

**Import Errors**:
```python
# Error: ModuleNotFoundError: No module named 'pandas'
# Fix: Add to requirements.txt
pandas>=2.0.0
```

**Missing create_agent()**:
```python
# Error: No create_agent() function found
# Fix: Add factory function
def create_agent():
    return SubstrateAgent(name="My Agent", model="gpt-4o-mini", prompt="...")
```

**Tool Registration Issues**:
```python
# Error: Tool not registered
# Fix: Register after creating agent
agent = SubstrateAgent(...)
agent.register_tool(my_tool)  # Don't forget this!
```

**Missing Tool Decorator**:
```python
# Error: Function is not a valid tool
# Fix: Add @tool decorator
from Daita.core.tools import tool

@tool  # Required!
async def my_tool(param: str) -> dict:
    return {"result": param}
```

**API Key Missing**:
```python
# Error: OpenAI API key not found
# Fix: Add to .env file
OPENAI_API_KEY=sk-...
```

**Async/Await Issues**:
```python
# Error: object dict can't be used in 'await' expression
# Fix: Make tool functions async
@tool
async def my_tool():  # Must be async
    return {"result": "value"}
```

### Project Structure During Development

```
my-project/
├── Daita-project.yaml    # Project config (don't modify during dev-loop)
├── agents/               # Your agent files (modify these)
│   ├── my_agent.py      # Agent being developed
│   └── helper.py        # Optional: helper modules
├── workflows/            # Workflow files
├── data/                 # Test data
│   └── test_input.json  # Custom test data
├── .env                  # API keys
├── requirements.txt      # Dependencies (add new ones here)
└── tests/               # Optional: Custom tests
```

### Watch Mode Behavior

**How Watch Mode Works**:
1. Runs initial test of all agents
2. Monitors file changes in `agents/` and `workflows/` directories
3. When file changes detected, waits 1 second for file write to complete
4. Re-runs tests automatically
5. Reports results (pass/fail, errors)
6. Continues watching until Ctrl+C

**What Triggers Retest**:
- Saving agent files (`.py` in `agents/`)
- Saving workflow files (`.py` in `workflows/`)
- Modifying `requirements.txt` (may need manual pip install)
- Modifying `.env` file (API key changes)

**What Doesn't Trigger Retest**:
- Changing `Daita-project.yaml` (config changes need manual retest)
- Modifying files outside `agents/` and `workflows/`
- Saving files in `data/` directory

### Development Workflow

**Typical Flow**:
1. Start with failing or incomplete agent
2. Run `Daita test --watch` to start watch mode
3. Make code changes (manually or via Claude)
4. Save file → watch mode detects → auto-retests
5. See results → diagnose → fix → repeat
6. When tests pass, stop watch mode
7. Deploy with `/ship` command

**Best Practices**:
- Fix one issue at a time (don't batch changes)
- Wait for test results before making next change
- Read error messages carefully (they're usually accurate)
- Test locally before deploying to production
- Keep test iterations fast (optimize slow tools)

### Error Diagnosis Strategy

**When test fails, check in order**:
1. **Import errors** → Add to `requirements.txt`
2. **Syntax errors** → Fix Python syntax
3. **create_agent() missing** → Add factory function
4. **Tool registration** → Call `agent.register_tool()`
5. **Tool decorator** → Add `@tool` decorator
6. **API keys** → Add to `.env` file
7. **Tool execution** → Debug tool logic
8. **Agent logic** → Adjust prompt or tool implementation

### Performance Tips

**Keep Tests Fast**:
- Use smaller models for development (`gpt-4o-mini` vs `gpt-4`)
- Mock external API calls when possible
- Cache results that don't change
- Avoid loading large datasets in tools
- Use `run()` instead of `run_detailed()` when metadata not needed

**Optimize Tool Execution**:
```python
@tool
async def fast_tool(data: list) -> dict:
    '''Process data quickly'''
    # Bad: Loading large model every call
    # model = load_large_model()

    # Good: Use simple computation
    return {"count": len(data), "sum": sum(data)}
```

### Need More Info?

If you're unsure about agent patterns, tool implementation, or testing:
- Check project's `CLAUDE.md` for custom patterns
- Visit **https://docs.Daita-tech.io/development** for dev guides
- Read existing agent files in `agents/` directory
- Review `Daita-project.yaml` for configuration

---

## Command Instructions

You are helping the user develop their Daita agent in an iterative loop with real-time feedback. This creates a powerful pair-programming experience.

## Setup

1. **Verify project**:
   - Check we're in a Daita project directory
   - Run `Daita status` to see available agents/workflows
   - Ask which agent/workflow to focus on (or use all if not specified)

2. **Explain the workflow**:
   - Tell user you'll start watch mode that auto-reruns tests on file changes
   - You'll monitor for failures and suggest/apply fixes
   - They can make changes manually or ask you to make changes
   - Press Ctrl+C to stop the loop

## Start Watch Mode

3. **Run watch mode in background**:
   - Run `Daita test --watch` in the background
   - Explain that tests will auto-run whenever files change
   - You'll monitor the output for failures

4. **Initial test run**:
   - Wait for first test execution to complete
   - Report results:
     - If passing: Great! Waiting for changes...
     - If failing: Analyze errors and suggest fixes (see below)

## Development Loop

5. **Monitor for changes**:
   - Watch the test output continuously
   - When tests fail, jump into diagnostic mode
   - When tests pass, acknowledge and wait for next change

6. **Error diagnosis** (when tests fail):
   - Read the error message from test output
   - Identify the failing agent/workflow and specific error
   - Categorize the error:
     - **Import errors**: Missing dependencies
     - **Syntax errors**: Code problems
     - **Runtime errors**: Logic issues
     - **Test failures**: Unexpected behavior
     - **API errors**: Missing keys or rate limits

7. **Suggest fixes**:
   - Based on the error, propose specific code changes
   - Explain what's wrong and why your fix will work
   - Ask if user wants you to apply the fix or they'll do it manually

8. **Apply fixes** (if user agrees):
   - Read the failing file
   - Make the necessary changes using Edit tool
   - Explain what you changed
   - Wait for watch mode to detect change and re-run tests
   - Report new test results

9. **Iterate**:
   - Keep monitoring and fixing until tests pass
   - Provide encouragement and progress updates
   - Suggest improvements even when tests pass

## User Interactions During Loop

10. **Respond to user requests**:
    - User can ask you to make changes at any time
    - User can ask questions about the code
    - User can request you to add new features
    - Always apply changes and wait for test feedback

11. **Proactive suggestions**:
    - If you notice patterns in errors, suggest architectural improvements
    - If tests are slow, suggest optimizations
    - If code is repetitive, suggest refactoring
    - Share best practices relevant to what they're building

## Advanced Features

12. **Test with custom data**:
    - If user wants to test with different inputs, create `data/test_input.json`
    - Run `Daita test --data data/test_input.json` in watch mode instead
    - Update test data as needed

13. **Multi-agent development**:
    - If working on a workflow with multiple agents, test them individually first
    - Then test the full workflow integration
    - Help debug communication between agents

14. **Performance monitoring**:
    - Note test execution times
    - If tests get slower, investigate and suggest optimizations
    - Watch for memory issues or timeouts

## Stopping the Loop

15. **When user is done**:
    - They'll press Ctrl+C or ask to stop
    - Provide summary of what was accomplished:
      - What errors were fixed
      - What features were added
      - Current test status
      - Suggested next steps

16. **Next steps**:
    - Suggest running full test suite: `Daita test`
    - Recommend committing changes if using git
    - Suggest deploying if ready: use `/ship` command

## Error Handling Examples

**Import Error**:
```
Error: ModuleNotFoundError: No module named 'pandas'
Fix: Add 'pandas>=2.0.0' to requirements.txt and run pip install -r requirements.txt
```

**API Key Missing**:
```
Error: OpenAI API key not found
Fix: Add OPENAI_API_KEY to .env file with your API key
```

**Tool Execution Error**:
```
Error: calculate_stats() missing required argument 'data'
Fix: Update tool call in agent to pass data parameter
```

**Timeout**:
```
Error: Agent execution timed out after 120s
Fix: Increase timeout in AgentConfig or optimize tool execution
```

## Tips for Effectiveness

- **Be conversational**: This is pair programming, not just error fixing
- **Explain reasoning**: Help user learn, don't just fix silently
- **Stay focused**: Keep track of what's being worked on
- **Celebrate wins**: Acknowledge when tests pass or issues are resolved
- **Be patient**: Development is iterative, errors are normal
- **Think ahead**: Suggest improvements even when things work

**Important**:
- Always wait for test results after making changes
- Don't apply multiple fixes at once - iterate one change at a time
- Make sure user understands what you're doing and why
- Keep the feedback loop tight and fast
- Follow Daita agent development patterns
- Have fun - this should feel collaborative and productive!
