# Daita Claude Code Commands

Supercharge your Daita development workflow with intelligent Claude Code slash commands. These commands turn Claude into an AI DevOps engineer that understands your entire agent development lifecycle.

## What Are These Commands?

These are custom [Claude Code](https://claude.com/code) slash commands that provide intelligent, multi-step automation for common Daita workflows. Instead of running multiple CLI commands manually, you can ask Claude to handle entire workflows - from project setup to production deployment.

## Features

- **Intelligent Automation**: Multi-step workflows with smart error handling
- **Safety-First**: Pre-flight checks prevent production issues
- **Context-Aware**: Claude understands your project structure and configuration
- **Production-Ready**: Deploy, debug, and monitor with confidence
- **Pair Programming**: Iterative development with real-time feedback

## Available Commands

### `/quickstart` - Bootstrap New Projects
Create a complete Daita project from scratch with custom agents tailored to your use case.

**Usage:**
```
/quickstart "customer sentiment analyzer"
```

**What it does:**
- Asks for project name and use case
- Runs `Daita init` to create project structure
- Generates custom agent with relevant tools
- Sets up test data and configuration
- Runs first test to validate setup
- Shows project status and next steps

**Perfect for:** Starting new projects, creating POCs, onboarding new developers

---

### `/ship` - Deploy to Production Safely
Deploy your project to production with comprehensive safety checks and validation.

**Usage:**
```
/ship
```

**What it does:**
- Verifies you're in a Daita project
- Runs `Daita test` to validate locally
- Checks `Daita status` for configuration issues
- Verifies Daita_API_KEY is set
- Increments version in `Daita-project.yaml`
- Runs `Daita push production`
- Shows deployment status and webhook URLs
- Provides monitoring commands

**Perfect for:** Production deployments, version releases, CI/CD integration

---

### `/debug-deployment` - Debug Production Issues
Diagnose and fix production deployment issues with intelligent log analysis.

**Usage:**
```
/debug-deployment "my agent is timing out"
```

**What it does:**
- Fetches production logs via `Daita logs production`
- Analyzes error messages and stack traces
- Identifies root cause (missing keys, timeouts, import errors, etc.)
- Suggests specific fixes
- Can apply fixes and redeploy
- Provides prevention advice

**Perfect for:** Production debugging, error analysis, incident response

---

### `/setup-webhook` - Configure Webhook Integrations
Set up webhook integrations for external service triggers (GitHub, Slack, Stripe, etc.).

**Usage:**
```
/setup-webhook github-push for my data_processor agent
```

**What it does:**
- Asks which agent/workflow to trigger
- Suggests field mapping based on service type
- Updates `Daita-project.yaml` with webhook config
- Deploys to production
- Runs `Daita webhook list` to show webhook URL
- Provides test curl command
- Shows integration instructions for the service

**Perfect for:** GitHub webhooks, Slack integrations, payment processing, custom APIs

**Supported Services:**
- GitHub (push, pull request, issues)
- Slack (slash commands, events)
- Stripe (payments, subscriptions)
- Twilio (SMS, calls)
- Custom webhooks

---

### `/dev-loop` - Iterative Development Mode
Run watch mode with intelligent error detection and auto-fixing.

**Usage:**
```
/dev-loop
```

**What it does:**
- Starts `Daita test --watch` in background
- Monitors test output in real-time
- Detects test failures automatically
- Diagnoses errors and suggests fixes
- Applies fixes when you approve
- Waits for tests to re-run
- Iterates until tests pass
- Provides encouragement and progress updates

**Perfect for:** Active development, debugging, learning Daita, pair programming

---

## Installation

### Prerequisites

1. **Install Daita CLI:**
   ```bash
   pip install Daita-agents
   ```

2. **Install Claude Code:**
   - Get Claude Code from [claude.com/code](https://claude.com/code)
   - Or use the Claude desktop app with code mode enabled

### Install Commands

**Option 1: Clone this repository**
```bash
git clone https://github.com/your-org/Daita-claude-commands.git
cd Daita-claude-commands
cp -r .claude/commands ~/.claude/commands/Daita/
```

**Option 2: Manual installation**
1. Download the command files from this repository
2. Copy them to your Claude Code commands directory:
   - **macOS/Linux:** `~/.claude/commands/Daita/`
   - **Windows:** `%USERPROFILE%\.claude\commands\Daita\`

**Option 3: Project-specific installation**
```bash
# In your Daita project directory
mkdir -p .claude/commands
cd .claude/commands
curl -O https://raw.githubusercontent.com/your-org/Daita-claude-commands/main/.claude/commands/quickstart.md
curl -O https://raw.githubusercontent.com/your-org/Daita-claude-commands/main/.claude/commands/ship.md
curl -O https://raw.githubusercontent.com/your-org/Daita-claude-commands/main/.claude/commands/debug-deployment.md
curl -O https://raw.githubusercontent.com/your-org/Daita-claude-commands/main/.claude/commands/setup-webhook.md
curl -O https://raw.githubusercontent.com/your-org/Daita-claude-commands/main/.claude/commands/dev-loop.md
```

### Verify Installation

Open Claude Code and type `/` - you should see the Daita commands listed:
- `/quickstart`
- `/ship`
- `/debug-deployment`
- `/setup-webhook`
- `/dev-loop`

## Quick Start Examples

### Example 1: Create and Deploy a New Agent

```bash
# Start a new project
/quickstart "fraud detection for payments"

# Claude creates project, generates agent with fraud detection tools, runs tests

# Make some customizations to the agent
# ... edit code ...

# Deploy to production
/ship

# Claude runs tests, increments version, deploys, shows webhook URLs
```

### Example 2: Add Webhook to Existing Project

```bash
# Navigate to your project
cd my-Daita-project

# Set up a webhook for GitHub push events
/setup-webhook github-push for payment_processor

# Claude configures webhook, deploys, provides webhook URL and integration instructions
```

### Example 3: Debug Production Issue

```bash
# Something's failing in production
/debug-deployment "agent returning empty results"

# Claude fetches logs, identifies missing API key, suggests fix
# You approve, Claude adds key to .env and redeploys
```

### Example 4: Active Development with Watch Mode

```bash
# Working on a new feature
/dev-loop

# Claude starts watch mode
# You make code changes
# Tests fail - Claude suggests fix
# You approve - Claude applies fix
# Tests pass - Claude waits for next change
# Ctrl+C when done
```

## Environment Setup

These commands work best when you have:

### Local Development (Free)
```bash
# LLM API key for agent execution
export OPENAI_API_KEY='sk-...'
# or
export ANTHROPIC_API_KEY='sk-ant-...'
```

### Cloud Deployment (Requires Daita_API_KEY)
```bash
# Get your API key from Daita-tech.io
export Daita_API_KEY='Daita-...'
```

You can also store these in a `.env` file in your project root.

## Command Reference Table

| Command | Requires Project | Requires Daita_API_KEY | Time to Complete |
|---------|------------------|------------------------|------------------|
| `/quickstart` | No | No | 30-60 seconds |
| `/ship` | Yes | Yes | 1-3 minutes |
| `/debug-deployment` | Yes | Yes | 1-5 minutes |
| `/setup-webhook` | Yes | Yes | 1-2 minutes |
| `/dev-loop` | Yes | No (for local) | Ongoing |

## Tips for Best Results

### For `/quickstart`
- Be specific about your use case: "sentiment analysis for customer reviews" vs just "analyzer"
- Mention any specific integrations you need: "with PostgreSQL" or "using Stripe API"
- Claude will generate better tools and test data with more context

### For `/ship`
- Always run from your project root directory
- Let Claude increment the version (it suggests patch/minor/major)
- Review the deployment summary before proceeding

### For `/debug-deployment`
- Describe symptoms specifically: "timeouts after 2 minutes" vs "not working"
- Mention when it started: "after recent deployment" helps narrow down issues
- Let Claude reproduce locally when possible

### For `/setup-webhook`
- Know which service you're integrating (GitHub, Slack, etc.)
- Claude provides field mapping templates for common services
- Test with the provided curl command before connecting real services

### For `/dev-loop`
- Focus on one agent/workflow at a time
- Let Claude fix one issue at a time (don't rush)
- Use this for learning - Claude explains what it's doing

## Troubleshooting

### "Command not found"
- Verify files are in the correct `.claude/commands/` directory
- Restart Claude Code
- Check file permissions (should be readable)

### "Not in a Daita project"
- Run `Daita init` first, or navigate to existing project
- Verify `Daita-project.yaml` exists

### "Daita_API_KEY not found"
- Get API key from [Daita-tech.io](https://Daita-tech.io)
- Set environment variable: `export Daita_API_KEY='your-key'`
- Or add to `.env` file

### Commands are slow
- Normal for first run (Claude analyzes project)
- Subsequent runs are faster with context
- `/dev-loop` stays fast since it's watching continuously

### Deployment fails
- Check `Daita test` passes locally first
- Verify all API keys are in `.env` file
- Review `Daita-project.yaml` for configuration issues
- Use `/debug-deployment` for detailed diagnosis

## Community

- **Daita Documentation**: [docs.Daita-tech.io](https://docs.Daita-tech.io)
- **Claude Code Docs**: [claude.com/code/docs](https://claude.com/code/docs)

## License

MIT License - see [LICENSE](LICENSE) file for details

## Acknowledgments

Built with:
- [Claude Code](https://claude.com/code) by Anthropic

---

Get started: `Daita init my-project` or `/quickstart "your use case"`
