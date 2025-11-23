---
description: Run iterative development loop with watch mode and auto-fixing
---

You are helping the user develop their DAITA agent in an iterative loop with real-time feedback. This creates a powerful pair-programming experience.

## Setup

1. **Verify project**:
   - Check we're in a DAITA project directory
   - Run `daita status` to see available agents/workflows
   - Ask which agent/workflow to focus on (or use all if not specified)

2. **Explain the workflow**:
   - Tell user you'll start watch mode that auto-reruns tests on file changes
   - You'll monitor for failures and suggest/apply fixes
   - They can make changes manually or ask you to make changes
   - Press Ctrl+C to stop the loop

## Start Watch Mode

3. **Run watch mode in background**:
   - Run `daita test --watch` in the background
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
    - Run `daita test --data data/test_input.json` in watch mode instead
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
    - Suggest running full test suite: `daita test`
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
- Have fun - this should feel collaborative and productive!
