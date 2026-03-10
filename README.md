# Daita Claude Code Commands

Supercharge your Daita development workflow with intelligent Claude Code slash commands. These commands turn Claude into an AI DevOps engineer that understands your entire agent development lifecycle — from bootstrapping a project to debugging production.

## What Are These Commands?

Custom [Claude Code](https://claude.ai/claude-code) slash commands and skills that automate common Daita workflows. Instead of running multiple CLI commands manually or writing boilerplate from scratch, Claude handles entire workflows with context-aware intelligence.

## Available Commands

| Command | What it does | Requires project? | Requires DAITA_API_KEY? |
|---|---|---|---|
| `/quickstart` | Bootstrap a new Daita project | No | No |
| `/ship` | Deploy to production with safety checks | Yes | Yes |
| `/dev-loop` | Watch mode + auto-fixing pair programmer | Yes | No |
| `/setup-webhook` | Configure external service webhooks | Yes | Yes |
| `/debug-deployment` | Diagnose production issues from logs | Yes | Yes |
| `/add-plugin` | Add a database/API/service plugin to an agent | Yes | No |
| `/create-workflow` | Build a multi-agent workflow | Yes | No |
| `/add-provider` | Switch or add an LLM provider | Yes | No |
| `/setup-memory` | Add persistent memory to an agent | Yes | No |

## Available Skills

Skills are reusable prompts invoked standalone or from within commands:

| Skill | What it generates |
|---|---|
| `/agent-scaffold` | Complete agent file from a description |
| `/tool-scaffold` | `@tool` function with docstring and error handling |
| `/test-scaffold` | Full pytest test file for an agent |

## Installation

### Prerequisites

```bash
pip install daita-agents       # Install Daita CLI
```

Get Claude Code: [claude.ai/claude-code](https://claude.ai/claude-code)

### Option 1: Clone and use directly (recommended)

When you open this repo in Claude Code, all commands and skills are available immediately — no copying needed.

```bash
git clone https://github.com/daita-tech/claude-code.git
cd claude-code
# Open in Claude Code — commands available right away
```

### Option 2: Install globally (use from any project)

```bash
git clone https://github.com/daita-tech/claude-code.git
cp -r claude-code/.claude/commands/* ~/.claude/commands/
cp -r claude-code/.claude/skills/* ~/.claude/skills/
```

### Option 3: Install per-project

```bash
# In your Daita project directory
mkdir -p .claude/commands .claude/skills
cp -r /path/to/claude-code/.claude/commands/* .claude/commands/
cp -r /path/to/claude-code/.claude/skills/* .claude/skills/
```

### Verify installation

Open Claude Code and type `/` — you should see the Daita commands listed.

## Quick Start Examples

### Start a new project

```
/quickstart "fraud detection agent for payment transactions"
```

Claude will ask about your LLM provider preference, scaffold a project with relevant tools, set up test data, and run the first test.

### Deploy to production

```
/ship
```

Claude runs tests, checks API keys, increments the version, deploys, and shows webhook URLs and monitoring commands.

### Add a database

```
/add-plugin "connect my agent to PostgreSQL"
```

Claude adds the PostgreSQLPlugin, writes tools to query it, updates your `.env`, and tests the connection.

### Debug a production issue

```
/debug-deployment "agent returning empty results since this morning"
```

Claude fetches logs, traces the execution, identifies the root cause, proposes a fix, and redeploys.

### Set up a GitHub webhook

```
/setup-webhook github push events for my code_reviewer agent
```

Claude configures the field mapping in `daita-project.yaml`, deploys, and gives you the webhook URL and GitHub setup instructions.

### Switch to Claude for better reasoning

```
/add-provider "switch to Anthropic for my analyst agent"
```

Claude updates the agent to use `claude-3-5-sonnet`, checks your API key, and tests the change.

### Build a multi-agent pipeline

```
/create-workflow "researcher agent feeds into a writer agent"
```

Claude designs the pipeline, scaffolds both agents, wires up the Workflow connections, and tests it end to end.

## Environment Setup

```bash
# LLM providers — add whichever you use
export OPENAI_API_KEY='sk-...'
export ANTHROPIC_API_KEY='sk-ant-...'
export GEMINI_API_KEY='...'

# Required for cloud deployment
export DAITA_API_KEY='daita-...'    # Get from daita-tech.io
```

Or add these to a `.env` file in your project root.

## Troubleshooting

**"Command not found"** — Verify files are in `.claude/commands/`. Restart Claude Code.

**"Not in a Daita project"** — Run `daita init` first, or navigate to your project directory.

**"DAITA_API_KEY not found"** — Get your key at daita-tech.io and run `export DAITA_API_KEY='daita-...'`.

**Tests are slow on first run** — Claude is reading your project structure. Subsequent runs in the same session are faster.

## Resources

- **Daita Documentation**: [docs.daita-tech.io](https://docs.daita-tech.io)
- **Claude Code Docs**: [claude.ai/claude-code](https://claude.ai/claude-code)
- **Issues**: [github.com/daita-tech/claude-code/issues](https://github.com/daita-tech/claude-code/issues)

## License

Apache 2.0 — see [LICENSE](LICENSE)
