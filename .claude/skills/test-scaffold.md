---
description: Generate a pytest test file for a Daita agent following the project's test conventions
---

Generate a complete test file for a Daita agent. Read the agent file first, then produce tests that cover both the tool functions and the agent's end-to-end behavior.

## Before generating

Read the agent file at `agents/[agent-name].py`. Understand:
- What tools are defined and what they do
- What inputs/outputs they expect
- What the agent's overall purpose is
- Whether any plugins are used

Also check if `tests/` directory exists and look at existing test files to match the project's conventions.

## Test file structure

Generate `tests/test_[agent-name].py`:

```python
"""Tests for [AgentName] agent."""
import pytest
import asyncio
from unittest.mock import AsyncMock, MagicMock, patch
from daita import Agent, tool
from daita.llm.mock import MockLLMProvider

# Import the tools and agent from the agent file
from agents.[agent_name] import [tool1], [tool2], agent


# ── Unit Tests: Tool Functions ─────────────────────────────────────────────

class TestTools:
    """Unit tests for individual tool functions.
    These run without any LLM or external service calls.
    """

    @pytest.mark.asyncio
    async def test_[tool1]_success(self):
        """[Tool1] returns correct result for valid input."""
        result = await [tool1]([valid_params])

        assert result["[expected_key]"] == [expected_value]
        assert "error" not in result

    @pytest.mark.asyncio
    async def test_[tool1]_with_edge_case(self):
        """[Tool1] handles [edge case description] gracefully."""
        result = await [tool1]([edge_case_params])

        assert result is not None
        # assert specific behavior for edge case

    @pytest.mark.asyncio
    async def test_[tool1]_error_handling(self):
        """[Tool1] returns error dict instead of raising on failure."""
        # Test with invalid input that should cause a handled error
        result = await [tool1]([invalid_params])

        assert "error" in result or result.get("success") is False


# ── Integration Tests: Agent Execution ────────────────────────────────────

class TestAgentWithMock:
    """Integration tests using MockLLMProvider.
    Tests agent behavior without real LLM API calls.
    """

    @pytest.fixture
    def mock_agent(self):
        """Create a fresh agent instance with mock LLM for testing."""
        test_agent = Agent(
            name="[agent-name]",
            llm_provider="mock",           # Uses MockLLMProvider
            model="mock",
            tools=[tool1, tool2],
            prompt=agent.prompt            # Reuse the real prompt
        )
        return test_agent

    @pytest.mark.asyncio
    async def test_agent_responds_to_basic_prompt(self, mock_agent):
        """Agent runs without error on a basic request."""
        result = await mock_agent.run("[realistic test prompt]")

        assert result is not None
        assert isinstance(result, str)
        assert len(result) > 0

    @pytest.mark.asyncio
    async def test_agent_run_detailed_returns_metadata(self, mock_agent):
        """run_detailed returns result with cost and timing metadata."""
        result = await mock_agent.run_detailed("[test prompt]")

        assert "result" in result
        assert "processing_time_ms" in result


# ── Live Tests: Real LLM ───────────────────────────────────────────────────

@pytest.mark.requires_llm
@pytest.mark.integration
class TestAgentLive:
    """Live tests that call the real LLM API.
    Only run these locally with API keys set.
    Skip in CI unless explicitly configured.
    """

    @pytest.mark.asyncio
    async def test_agent_live_basic_run(self):
        """Agent successfully completes a real task end-to-end."""
        result = await agent.run("[realistic test prompt that exercises the tools]")

        assert result is not None
        assert len(result) > 10  # Got a real response

    @pytest.mark.asyncio
    async def test_agent_live_uses_tools(self):
        """Agent calls the expected tools when given a relevant prompt."""
        events = []

        def capture_events(event):
            events.append(event)

        result = await agent.run(
            "[prompt that should trigger tool usage]",
            on_event=capture_events
        )

        tool_calls = [e for e in events if hasattr(e, 'type') and str(e.type) == 'EventType.TOOL_CALL']
        assert len(tool_calls) > 0, "Expected agent to call at least one tool"


# ── Fixtures and Helpers ──────────────────────────────────────────────────

@pytest.fixture
def sample_[domain]_data():
    """Realistic sample data for testing [agent domain]."""
    return {
        "[field1]": "[realistic_value]",
        "[field2]": "[realistic_value]",
    }
```

## Generation rules

### For each tool, generate:

1. **Happy path test** — normal input, verify expected output structure
2. **Edge case test** — empty input, boundary values, unusual but valid input
3. **Error handling test** — invalid input that should be caught and returned as an error dict (not raised)

### For the agent integration tests, generate:

1. **Mock-based test** — using `llm_provider="mock"` so no API key is needed
2. **Metadata test** — verify `run_detailed()` returns expected fields
3. **Live test** (marked `@pytest.mark.requires_llm`) — realistic end-to-end scenario

### Naming conventions:

- Test class names: `TestTools`, `TestAgentWithMock`, `TestAgentLive`
- Test method names: `test_[tool]_[scenario]` or `test_agent_[scenario]`
- Fixture names: `mock_agent`, `sample_[domain]_data`

### Markers to use:

```python
@pytest.mark.asyncio          # for any async test
@pytest.mark.requires_llm     # for tests that need a real API key
@pytest.mark.integration      # for tests that call external services
@pytest.mark.slow             # for tests that take more than a few seconds
```

### What NOT to test:

- Don't test internal Daita framework behavior (that's the framework's responsibility)
- Don't test that the LLM gives a specific exact answer (LLM outputs are non-deterministic)
- Don't create mocks for everything — test real tool logic with real computation

### After generating:

Show the user how to run the tests:
```bash
# Run unit tests only (no API key needed)
pytest tests/test_[agent-name].py -m "not requires_llm" -v

# Run all tests including live LLM tests (needs API key)
pytest tests/test_[agent-name].py -v

# Run just the live tests
pytest tests/test_[agent-name].py -m requires_llm -v
```

Also confirm whether a `conftest.py` with asyncio mode config exists:
```python
# conftest.py (if needed)
import pytest

@pytest.fixture(scope="session")
def event_loop_policy():
    return asyncio.DefaultEventLoopPolicy()
```

And whether `pytest.ini` or `pyproject.toml` has `asyncio_mode = "auto"` configured.
