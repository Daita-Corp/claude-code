---
description: Give your Daita agent persistent memory so it remembers information across runs
---

## Daita Framework Context

You are adding **persistent memory** to a Daita agent using the Memory plugin. Memory lets agents store and retrieve semantic information across separate runs — they can remember user preferences, past decisions, accumulated knowledge, and more.

### Two Types of Memory in Daita

**1. Conversation History** — remembers the message flow within a session:
```python
from daita import Agent
from daita.agents.conversation import ConversationHistory

# Conversation history is built into Agent — just pass a conversation_id
agent = Agent(
    name="assistant",
    llm_provider="openai",
    model="gpt-4o",
    tools=[...],
    prompt="You are a helpful assistant with memory of our conversation."
)

# Pass the same conversation_id across calls to maintain context
result = await agent.run("My name is Alice", conversation_id="user-alice-123")
result = await agent.run("What's my name?", conversation_id="user-alice-123")
# Agent remembers: "Your name is Alice"
```

**2. Semantic Memory Plugin** — stores and retrieves information by meaning across sessions:
```python
from daita import Agent, tool
from daita.plugins.memory import MemoryPlugin

# Initialize memory backend
memory = MemoryPlugin(uri=os.environ["MEMORY_URI"])  # e.g., "postgresql://..." or local path

agent = Agent(
    name="assistant",
    llm_provider="openai",
    model="gpt-4o",
    tools=[remember_fact, recall_facts, forget_fact],
    prompt="You have persistent memory. Store important information and recall it when relevant."
)
agent.add_plugin(memory)
```

### Memory Plugin Tools Pattern

The agent doesn't use memory automatically — you write tools that wrap the memory operations, giving the agent full control over what to store and retrieve:

```python
from daita import tool
from daita.plugins.memory import MemoryPlugin
import os

memory = MemoryPlugin(uri=os.environ.get("MEMORY_URI", "./memory.db"))

@tool
async def remember(key: str, value: str, context: str = "") -> dict:
    """Store a piece of information in persistent memory for later recall.

    Args:
        key: A short identifier or label for this memory (e.g., 'user_preference_color')
        value: The information to store
        context: Optional additional context about why this is being stored
    Returns:
        Confirmation that the memory was saved
    """
    await memory.store(key=key, value=value, metadata={"context": context})
    return {"stored": True, "key": key}

@tool
async def recall(query: str, limit: int = 5) -> dict:
    """Search persistent memory for relevant stored information.

    Args:
        query: A natural language description of what you're looking for
        limit: Maximum number of results to return (default: 5)
    Returns:
        List of relevant memories matching the query
    """
    results = await memory.search(query=query, limit=limit)
    return {"memories": results, "count": len(results)}

@tool
async def forget(key: str) -> dict:
    """Remove a specific piece of stored information from memory.

    Args:
        key: The key of the memory to remove
    Returns:
        Confirmation that the memory was removed
    """
    await memory.delete(key=key)
    return {"deleted": True, "key": key}
```

### Memory URI Options

```bash
# Local file (development)
MEMORY_URI=./memory.db

# PostgreSQL with pgvector (production — semantic search)
MEMORY_URI=postgresql://user:password@host:5432/dbname

# Remote memory service
MEMORY_URI=https://memory.daita-tech.io/your-org-id
```

### When to Use Conversation History vs. Memory Plugin

| Need | Solution |
|---|---|
| Remember messages within a single session | `conversation_id` parameter on `agent.run()` |
| Remember user preferences across sessions | Memory plugin with `remember` / `recall` tools |
| Store a growing knowledge base | Memory plugin with semantic search |
| Accumulate facts over many runs | Memory plugin |
| Agent collaboration (share knowledge) | Memory plugin with shared URI |

### Example: Personal Assistant with Memory

```python
from daita import Agent, tool
from daita.plugins.memory import MemoryPlugin
import os

memory = MemoryPlugin(uri=os.environ.get("MEMORY_URI", "./assistant_memory.db"))

@tool
async def remember_preference(category: str, preference: str) -> dict:
    """Remember a user preference or important personal detail.

    Args:
        category: Category of preference (e.g., 'food', 'schedule', 'communication_style')
        preference: The preference to remember
    """
    await memory.store(key=f"preference_{category}", value=preference)
    return {"remembered": True}

@tool
async def recall_about_user(topic: str) -> dict:
    """Recall what we know about the user regarding a topic.

    Args:
        topic: What aspect of the user you want to recall (e.g., 'dietary restrictions')
    """
    results = await memory.search(query=topic, limit=3)
    return {"recalled": results}

agent = Agent(
    name="personal_assistant",
    llm_provider="anthropic",
    model="claude-3-5-sonnet-20241022",
    tools=[remember_preference, recall_about_user],
    prompt="""You are a personal assistant with persistent memory.
    When you learn something important about the user (preferences, constraints, goals),
    store it using remember_preference. When answering questions or giving advice,
    first recall relevant context using recall_about_user."""
)
agent.add_plugin(memory)
```

### Environment Variables

```bash
# .env
MEMORY_URI=./memory.db              # Local for development
# MEMORY_URI=postgresql://...       # PostgreSQL for production
```

---

## Command Instructions

You are adding persistent memory to a Daita agent. The right approach depends on what the user wants the agent to remember and for how long.

## Step 1 — Understand the memory use case

Ask:
- What should the agent remember? (user preferences, past events, knowledge, decisions?)
- Should it persist across sessions or just within a session?
- How many things needs to be remembered? (a few key facts vs. a large knowledge base)
- Is this for a single user or multiple users? (affects how keys/namespacing works)

This determines whether conversation history alone is sufficient, or whether the Memory plugin is needed.

## Step 2 — Recommend the right approach

**Conversation history only** (simpler — good for session-scoped memory):
- The agent will remember everything said in the current run
- Add `conversation_id` parameter to `agent.run()` calls
- No plugin needed

**Memory plugin** (persistent across sessions):
- Use when the agent needs to remember things between separate invocations
- Requires the MemoryPlugin and tools that wrap store/retrieve operations

If conversation history is sufficient, explain that and implement it (no plugin needed). Only add the plugin if the use case genuinely requires cross-session persistence.

## Step 3 — Read the current agent

Read the agent file from `agents/`. Understand what it currently does.

## Step 4a — If using conversation history only

Show how to pass `conversation_id`:
```python
result = await agent.run(user_message, conversation_id=f"user-{user_id}")
```

Update the agent's `prompt` to acknowledge that it has conversation memory.

## Step 4b — If using Memory plugin

1. Add the import and plugin initialization (with `os.environ` for the URI)
2. Write `remember`, `recall`, and optionally `forget` tools
3. Add tools to the `Agent(tools=[...])` list
4. Call `agent.add_plugin(memory)` after creating the agent
5. Update the `prompt` to tell the agent when and how to use its memory

Tailor the tool names and docstrings to the specific use case (e.g., `remember_customer_preference` instead of generic `remember`).

## Step 5 — Set up the MEMORY_URI

For development (local file):
- Add `MEMORY_URI=./memory.db` to `.env`

For production:
- Recommend PostgreSQL with pgvector for proper semantic search
- Add the URI to production secrets: `daita secrets`

## Step 6 — Test

```bash
daita test [agent-name]
```

Test that:
1. The agent can store a memory
2. The agent can retrieve it in a subsequent call
3. Memories persist between test runs (check the memory file/DB)

## Step 7 — Summary

Explain to the user:
- What memory approach was used and why
- What the agent will remember and how
- Where memories are stored (`MEMORY_URI`)
- For production: how to configure the memory backend (`daita secrets`)
- How to test that memory is working

**Important**: Make sure the tools' docstrings clearly tell the agent *when* to use memory — the agent decides whether to call `remember` or `recall` based on those docstrings. Vague docstrings lead to the agent either storing too much (every message) or too little (nothing useful).
