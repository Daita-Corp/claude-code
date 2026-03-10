---
description: Switch your agent to a different LLM provider or add multi-provider support
---

## Daita Framework Context

You are working with the **Daita AI Agents Framework**, which supports four LLM providers natively. You can switch providers or configure different agents to use different providers.

### Supported Providers

| Provider | `llm_provider` value | Env var | Example models |
|---|---|---|---|
| OpenAI | `"openai"` | `OPENAI_API_KEY` | `gpt-4o`, `gpt-4o-mini` |
| Anthropic | `"anthropic"` | `ANTHROPIC_API_KEY` | `claude-3-5-sonnet-20241022`, `claude-3-haiku-20240307`, `claude-opus-4-6` |
| Google Gemini | `"gemini"` | `GEMINI_API_KEY` | `gemini-1.5-pro`, `gemini-1.5-flash`, `gemini-2.0-flash` |
| xAI Grok | `"grok"` | `GROK_API_KEY` | `grok-beta`, `grok-2` |

### Switching Providers

Simply update `llm_provider` and `model` in the Agent constructor:

```python
# Before: OpenAI
agent = Agent(
    name="analyst",
    llm_provider="openai",
    model="gpt-4o-mini",
    tools=[analyze_data],
    prompt="You are a data analyst..."
)

# After: Anthropic
agent = Agent(
    name="analyst",
    llm_provider="anthropic",
    model="claude-3-5-sonnet-20241022",
    tools=[analyze_data],
    prompt="You are a data analyst..."
)
```

Tools and prompts work the same regardless of provider — the framework handles provider-specific formatting automatically.

### Provider Strengths

**OpenAI (GPT-4o, GPT-4o-mini)**
- Best default choice for most tasks
- Fast and cost-effective with `gpt-4o-mini`
- Strong function calling / tool use
- Good at following structured output instructions

**Anthropic (Claude)**
- Best for complex reasoning and analysis tasks
- Handles long documents and context extremely well
- More nuanced instruction following
- Great for writing, summarization, code review
- `claude-3-haiku` is fast and cheap for simple tasks

**Google Gemini**
- Best for multimodal tasks (images, documents, video)
- Very long context window (1M+ tokens on Pro)
- Good for high-throughput workloads with Flash
- Strong at structured data extraction

**xAI Grok**
- Integrated with real-time X/Twitter data
- Good for tasks requiring current events knowledge
- Newer model, rapidly improving

### Multiple Providers in One Project

Different agents in the same project can use different providers — this is a powerful pattern:

```python
# Fast, cheap agent for classification
classifier = Agent(
    name="classifier",
    llm_provider="openai",
    model="gpt-4o-mini",          # cheap and fast
    tools=[classify_intent],
    prompt="Classify the user's intent quickly and accurately."
)

# Deep reasoning agent for complex analysis
reasoner = Agent(
    name="reasoner",
    llm_provider="anthropic",
    model="claude-3-5-sonnet-20241022",   # best reasoning
    tools=[analyze_document, extract_insights],
    prompt="Perform deep analysis of complex documents."
)

# Long-context agent for document processing
doc_processor = Agent(
    name="doc_processor",
    llm_provider="gemini",
    model="gemini-1.5-pro",        # massive context window
    tools=[process_large_file],
    prompt="Process and extract information from large documents."
)
```

### Environment Variables

```bash
# .env file — add whichever providers you use
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=...
GROK_API_KEY=xai-...
```

For production deployment, add secrets with:
```bash
daita secrets
```

### Cost Considerations

| Use case | Recommended |
|---|---|
| Development / testing | `gpt-4o-mini` or `claude-3-haiku` — cheap and fast |
| Complex reasoning | `claude-3-5-sonnet` or `gpt-4o` |
| Long documents (>100K tokens) | `gemini-1.5-pro` |
| Production cost-sensitive | `gpt-4o-mini`, `gemini-1.5-flash`, or `claude-3-haiku` |
| Best quality | `claude-opus-4-6` or `gpt-4o` |

---

## Command Instructions

You are helping the user change the LLM provider for one or more of their Daita agents, or add support for multiple providers across their project.

## Step 1 — Understand the goal

Ask (or determine from context):
- Which agent(s) do they want to change?
- Are they switching entirely, or adding a new agent with a different provider?
- What's driving the change? (cost, quality, capabilities, long context, etc.)

Common scenarios:
- "Switch to Claude for better reasoning" → update `llm_provider` and `model`
- "Use Gemini for its long context window" → update and note the context advantage
- "Make it cheaper for development" → switch to a smaller model in same family
- "Use different providers for different agents" → configure each agent independently

## Step 2 — Read the current agent(s)

Read the agent file(s) in `agents/`. Note the current `llm_provider` and `model` values.

## Step 3 — Recommend the right provider and model

Based on their reason for switching, recommend a specific provider and model:
- If they want **better reasoning/analysis**: Anthropic `claude-3-5-sonnet-20241022`
- If they want **cheaper development**: OpenAI `gpt-4o-mini` or Anthropic `claude-3-haiku-20240307`
- If they want **long context**: Gemini `gemini-1.5-pro`
- If they want **production quality**: `claude-3-5-sonnet` or `gpt-4o`
- If they want **real-time knowledge**: Grok `grok-beta`

Explain the tradeoff briefly.

## Step 4 — Check the API key

Verify the required API key is set:
- `OPENAI_API_KEY` for OpenAI
- `ANTHROPIC_API_KEY` for Anthropic
- `GEMINI_API_KEY` for Gemini
- `GROK_API_KEY` for Grok

If it's not in `.env`, add a placeholder and tell them where to get the key.

## Step 5 — Update the agent file(s)

Make the minimal change: update `llm_provider` and `model` in the `Agent(...)` constructor. The tools and prompt don't need to change — they work across all providers.

If switching multiple agents to different providers, update each one.

## Step 6 — Note any prompt adjustments (if needed)

Different providers have slightly different instruction-following behaviors:
- **Anthropic**: Responds well to XML-style structure in prompts (`<instructions>`, `<examples>`)
- **OpenAI**: Works well with numbered lists and clear sections
- **Gemini**: Good with detailed explicit instructions

If the current prompt is highly tuned for the old provider, suggest minor adjustments. But don't rewrite working prompts without reason.

## Step 7 — Update requirements.txt (if needed)

Daita handles provider SDKs internally — no provider-specific packages needed in `requirements.txt` unless using direct SDK calls.

## Step 8 — Test

```bash
daita test [agent-name]
```

Verify the agent works with the new provider. Watch for:
- API key errors → key not set correctly
- Model name errors → verify exact model ID
- Behavior differences → prompt might need minor tuning

## Step 9 — For production: update secrets

```bash
daita secrets
```

Add the new provider's API key to production secrets. Local `.env` is not enough for deployed agents.

## Step 10 — Summary

Tell the user:
- What was changed and why the new provider suits their use case
- What API key they need to get/set
- Any behavior differences to expect
- How to run `/ship` when ready to deploy the change

**Tip**: If they're not sure which provider to use, suggest starting with `gpt-4o-mini` (fast, cheap, reliable) and only upgrading to a more powerful model if the quality isn't sufficient for their use case.
