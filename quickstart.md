---
description: Bootstrap a new Daita project from scratch with intelligent setup
---

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

At the end, provide a summary of:
- What was created
- How to continue development locally (free)
- How to deploy to production when ready (with Daita_API_KEY)
