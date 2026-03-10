---
description: Add a data source plugin (database, vector DB, API, messaging) to an existing Daita agent
---

## Daita Framework Context

You are adding a **plugin** to an existing Daita agent. Plugins give agents native access to databases, vector stores, cloud storage, APIs, and messaging services without manual connection boilerplate.

### How Plugins Work

Plugins are added to an agent with `agent.add_plugin(plugin)`. Once added, the agent can access the plugin's functionality through its tools:

```python
from daita import Agent, tool
from daita.plugins import PostgreSQLPlugin

pg = PostgreSQLPlugin(
    host=os.environ["DB_HOST"],
    port=5432,
    database=os.environ["DB_NAME"],
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASSWORD"]
)

agent = Agent(
    name="data_analyst",
    llm_provider="openai",
    model="gpt-4o",
    tools=[query_database, summarize_results],
    prompt="You analyze data from our PostgreSQL database..."
)
agent.add_plugin(pg)
```

### Available Plugins

**Relational Databases**:
```python
from daita.plugins import PostgreSQLPlugin, MySQLPlugin

pg = PostgreSQLPlugin(host=..., database=..., user=..., password=...)
mysql = MySQLPlugin(host=..., database=..., user=..., password=...)
```

**NoSQL & Document Stores**:
```python
from daita.plugins import MongoDBPlugin

mongo = MongoDBPlugin(uri=os.environ["MONGODB_URI"], database="mydb")
```

**Data Warehouses**:
```python
from daita.plugins import SnowflakePlugin

snow = SnowflakePlugin(
    account=os.environ["SNOWFLAKE_ACCOUNT"],
    user=os.environ["SNOWFLAKE_USER"],
    password=os.environ["SNOWFLAKE_PASSWORD"],
    warehouse="COMPUTE_WH",
    database="MY_DB"
)
```

**Vector Databases** (for semantic search, RAG):
```python
from daita.plugins import PineconePlugin, ChromaPlugin, QdrantPlugin

pinecone = PineconePlugin(api_key=os.environ["PINECONE_API_KEY"], index_name="my-index")
chroma = ChromaPlugin(collection_name="my-docs", persist_directory="./chroma_db")
qdrant = QdrantPlugin(url=os.environ["QDRANT_URL"], collection_name="my-collection")
```

**Search & Analytics**:
```python
from daita.plugins import ElasticsearchPlugin

es = ElasticsearchPlugin(hosts=["https://my-cluster:9200"], api_key=os.environ["ES_API_KEY"])
```

**Cloud Storage**:
```python
from daita.plugins import S3Plugin

s3 = S3Plugin(
    bucket_name=os.environ["S3_BUCKET"],
    region="us-east-1",
    aws_access_key_id=os.environ["AWS_ACCESS_KEY_ID"],
    aws_secret_access_key=os.environ["AWS_SECRET_ACCESS_KEY"]
)
```

**Messaging & Notifications**:
```python
from daita.plugins import SlackPlugin

slack = SlackPlugin(token=os.environ["SLACK_BOT_TOKEN"])
```

**Graph Databases**:
```python
from daita.plugins import Neo4jPlugin

neo4j = Neo4jPlugin(
    uri=os.environ["NEO4J_URI"],
    user=os.environ["NEO4J_USER"],
    password=os.environ["NEO4J_PASSWORD"]
)
```

**HTTP APIs**:
```python
from daita.plugins import RESTPlugin

api = RESTPlugin(
    base_url="https://api.example.com",
    headers={"Authorization": f"Bearer {os.environ['API_TOKEN']}"}
)
```

**Web Search**:
```python
from daita.plugins import WebSearchPlugin

search = WebSearchPlugin(api_key=os.environ["TAVILY_API_KEY"])
```

**Email**:
```python
from daita.plugins import EmailPlugin

email = EmailPlugin(
    smtp_host="smtp.gmail.com",
    smtp_port=587,
    username=os.environ["EMAIL_USER"],
    password=os.environ["EMAIL_PASSWORD"]
)
```

**Model Context Protocol (MCP)**:
```python
from daita.plugins import MCPPlugin

mcp = MCPPlugin(server_url=os.environ["MCP_SERVER_URL"])
```

### Writing Tools That Use Plugin Data

Tools access plugin data through standard Python — the plugin handles connection management:

```python
@tool
async def query_customers(customer_id: str) -> dict:
    """Look up a customer record by ID and return their details.

    Args:
        customer_id: The unique customer identifier
    Returns:
        dict with customer details or error message
    """
    try:
        results = await pg.execute_query(
            "SELECT * FROM customers WHERE id = %s",
            params=[customer_id]
        )
        return {"found": True, "customer": results[0] if results else None}
    except Exception as e:
        return {"found": False, "error": str(e)}
```

### Environment Variables Best Practice

Never hardcode credentials. Use environment variables:
```python
import os
from daita.plugins import PostgreSQLPlugin

pg = PostgreSQLPlugin(
    host=os.environ.get("DB_HOST", "localhost"),
    database=os.environ["DB_NAME"],      # Required — will raise if missing
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASSWORD"]
)
```

For production, use `daita secrets` to store secrets (not `.env` file):
```bash
daita secrets                # List/manage production secrets
```

---

## Command Instructions

You are adding a data source plugin to an existing Daita agent. Make the integration clean, use environment variables for all credentials, and write tools that actually use the plugin.

## Step 1 — Identify the plugin

Ask (or determine from context):
- What data source do they want to connect? (PostgreSQL, Slack, S3, Pinecone, etc.)
- Which agent should get access to it?
- What should the agent be able to do with it? (query, write, search, send messages, etc.)

## Step 2 — Read the current agent

Read the agent file from `agents/`. Understand what it currently does — you'll be adding to it, not replacing it.

## Step 3 — Determine credentials and environment variables

Based on the plugin type, identify what credentials are needed. For each:
- Name the environment variable (e.g., `DB_HOST`, `SLACK_BOT_TOKEN`)
- Check if it's already in `.env`
- If not, add it as a placeholder with a comment explaining what to put there

Never hardcode credentials in the agent file.

## Step 4 — Add the plugin

Update the agent file:

1. Add the import at the top
2. Initialize the plugin with `os.environ` references
3. Call `agent.add_plugin(plugin)` after creating the agent
4. Write 1-3 tools that use the plugin for the agent's stated purpose
5. Update the agent's `prompt` to mention the new capability

## Step 5 — Update .env.example (or create it)

If there's an `.env.example` or `.env` template in the project, add the new required variables with placeholder values and comments explaining how to get them.

## Step 6 — Update requirements.txt

Some plugins have optional dependencies. Check if the plugin needs additional packages and add them to `requirements.txt`. Common extras:
- PostgreSQL: `psycopg2-binary>=2.9`
- MongoDB: `pymongo>=4.0`
- Snowflake: `snowflake-connector-python>=3.0`
- Pinecone: `pinecone-client>=3.0`
- AWS S3: `boto3>=1.26`
- Slack: `slack-sdk>=3.0`
- Elasticsearch: `elasticsearch>=8.0`

## Step 7 — Test

```bash
daita test [agent-name]
```

If the database/service is available locally, verify the tools actually work. If not (e.g., production-only database), test with mock data and note that full testing requires the real service.

## Step 8 — Summary

Tell the user:
- What plugin was added
- What new tools the agent has
- What environment variables need to be set
- For cloud deployment: run `daita secrets` to add secrets to production (`.env` alone isn't enough)

**Important**: Always use `os.environ` for credentials. Always test after adding. If the service isn't available locally, explain what the user needs to set up before the tools will work.
