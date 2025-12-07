# Jira Story Points Scripts for GenAI Coding Agents

When working with GenAI coding agents like Claude Code, you often want them to interact with external systems as part of their workflow. For Jira integration, I needed a way for the agent to read and update story point estimates on tickets—a common task when breaking down work or adjusting estimates after implementation.

## The Problem: No Atlassian CLI Support

I could not find a way to accomplish story point read/update operations through the Atlassian CLI. The official Atlassian CLI tools focus on other operations, and while there are third-party alternatives, I wanted something lightweight and portable that could easily be invoked by a GenAI agent.

## The Solution: Custom Bash Scripts

Working with Claude Code, I created two simple bash scripts that use the Jira REST API directly:

- **[jira-read-storypoints.sh](https://dev.nathanfox.net/scripts/jira-storypoints/jira-read-storypoints.sh)** - Read story points for one or more tickets
- **[jira-update-storypoints.sh](https://dev.nathanfox.net/scripts/jira-storypoints/jira-update-storypoints.sh)** - Update story points for one or more tickets

Full documentation and setup instructions: [dev.nathanfox.net/scripts/jira-storypoints/](https://dev.nathanfox.net/scripts/jira-storypoints/)

## Script Features

Both scripts share common characteristics that make them suitable for GenAI agent interaction:

- **Clear error messages**: When environment variables are missing, the scripts provide helpful setup instructions
- **Batch operations**: Process multiple tickets in a single invocation
- **Structured output**: Consistent, parseable output format for agent consumption
- **Exit codes**: Return appropriate exit codes (success/failure count) for programmatic handling

### Reading Story Points

```bash
# Single ticket
jira-read-storypoints.sh PROJ-123
# Output: PROJ-123: 5 points - Implement user authentication

# Multiple tickets
jira-read-storypoints.sh PROJ-123 PROJ-124 PROJ-125
```

### Updating Story Points

```bash
# Single ticket
jira-update-storypoints.sh PROJ-123 3

# Multiple tickets (pairs of ticket and points)
jira-update-storypoints.sh PROJ-123 3 PROJ-124 5 PROJ-125 8
```

## Environment Setup

The scripts require four environment variables:

```bash
export JIRA_URL='https://mycompany.atlassian.net'
export JIRA_EMAIL='your-email@example.com'
export JIRA_API_TOKEN='your_api_token_here'
export JIRA_STORY_POINTS_FIELD='customfield_XXXXX'
```

### Finding Your Story Points Custom Field ID

Story points in Jira are stored as a custom field, not a standard field. The field ID (e.g., `customfield_10016`) is unique to each Jira instance since IDs are assigned sequentially as fields are created.

**Option 1: Jira Admin UI**

1. Go to Jira Settings (gear icon) → Issues → Custom fields
2. Find "Story Points" or "Story point estimate" in the list
3. Click on it to view details
4. The field ID is in the URL: `.../customFields/configure?fieldId=customfield_XXXXX`

**Option 2: API Query**

```bash
curl -s -u "your-email@example.com:$JIRA_API_TOKEN" \
  "$JIRA_URL/rest/api/3/field" | jq '.[] | select(.name | test("story point"; "i")) | {name, id}'
```

This returns something like:

```json
{
  "name": "Story point estimate",
  "id": "customfield_10016"
}
```

**Option 3: Inspect a Ticket**

Query any ticket that has story points set and look for the value among custom fields:

```bash
curl -s -u "your-email@example.com:$JIRA_API_TOKEN" \
  "$JIRA_URL/rest/api/3/issue/PROJ-123" | jq '.fields | to_entries[] | select(.key | startswith("customfield_"))'
```

## GenAI Agent Integration

These scripts were specifically designed to be invoked by GenAI coding agents. A typical workflow might look like:

1. Agent receives a task: "Estimate PROJ-456 and update Jira"
2. Agent reviews the ticket requirements and codebase
3. Agent calls `jira-update-storypoints.sh PROJ-456 5` to set the estimate
4. Agent confirms the update with `jira-read-storypoints.sh PROJ-456`

The clear output format makes it easy for agents to verify operations succeeded and extract relevant information from the response.

For a complete workflow using these scripts for agent-based estimation, see [Using GenAI Agents to Estimate Story Points](https://www.nathanfox.net/p/genai-agent-story-point-estimation).

## Why Bash Scripts?

Bash scripts work well for GenAI agent tooling because:

- **Universal availability**: Present on virtually all development machines
- **No dependencies**: Beyond `curl` and `jq`, no additional packages needed
- **Easy invocation**: Simple command-line interface that agents understand
- **Portable**: Copy to any machine with the required environment variables
- **Transparent**: Easy to audit what the script does
