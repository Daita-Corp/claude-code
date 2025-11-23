---
description: Deploy your DAITA project to production with safety checks
---

You are helping the user deploy their DAITA project to production. This is a critical operation, so follow these safety checks:

## Pre-flight Checks

1. **Verify we're in a DAITA project**:
   - Check for `daita-project.yaml` in current directory or parent directories
   - If not found, tell user they need to be in a DAITA project directory

2. **Run local tests first**:
   - Run `daita test` to validate all agents and workflows work locally
   - If tests fail, STOP and help fix the issues before deploying
   - Show test results clearly

3. **Check project status**:
   - Run `daita status` to review the project configuration
   - Check for any configuration issues or missing files
   - Verify that agents and workflows are properly configured

4. **Verify API key**:
   - Check if `DAITA_API_KEY` is set in environment
   - If not set, explain how to get one from daita-tech.io and STOP
   - Don't proceed without a valid API key

5. **Check version**:
   - Read the current version from `daita-project.yaml`
   - Ask user if they want to increment the version (suggest patch, minor, or major based on changes)
   - If yes, update the version in `daita-project.yaml`

## Deployment

6. **Deploy to production**:
   - Run `daita push production --verbose` to deploy
   - Monitor the output for any errors
   - If deployment fails, help diagnose and fix the issue

7. **Verify deployment**:
   - Run `daita deployments list` to confirm the deployment succeeded
   - Show the deployment ID and timestamp

8. **Show webhook URLs** (if applicable):
   - Run `daita webhook list` to display webhook URLs
   - Explain how to use them for integrations

## Post-deployment

9. **Provide summary**:
   - Summarize what was deployed (agents, workflows, version)
   - Show the deployment ID for reference
   - Provide commands to monitor the deployment:
     - `daita logs production` - View execution logs
     - `daita status` - Check deployment status
     - `daita deployments list` - View deployment history

10. **Next steps**:
    - Suggest testing the deployed agents remotely
    - Explain how to view logs and monitor performance
    - Mention rollback option if needed: `daita deployments rollback <deployment-id>`

**Important**:
- NEVER deploy if tests fail
- ALWAYS increment version for tracking
- Provide clear error messages and solutions
- Make the user feel confident about the deployment
