# Using GenAI Agents to Estimate Story Points

Story point estimation often drifts from its intended purpose. I've found myself and teams I've worked with defaulting to "one story point equals one day of work" rather than treating points as a relative measure of complexity compared to a baseline story. What if a GenAI coding agent could bring the rigor back to estimation by actually analyzing complexity and comparing to historical work?

I've been using Claude Code to estimate story points with surprisingly accurate results. The process leverages the agent's ability to read external systems, analyze code, and reason about complexity—exactly what a developer does mentally during estimation.

## The Estimation Workflow

The workflow involves several steps that mirror how an experienced developer would approach estimation:

### 1. Read the Work Ticket

First, the agent reads the work ticket to understand the requirements. This workflow assumes you have CLI tools that allow the agent to interact with your issue tracker (Jira, GitHub, Linear, Azure DevOps, or whichever system you use):

```
Read PROJ-456 and summarize the requirements
```

The agent uses these CLI tools to pull ticket descriptions, acceptance criteria, and any linked documentation directly from your issue tracker.

### 2. Plan the Implementation

Next, the agent creates an implementation plan:

```
Based on PROJ-456, create a plan for what needs to be done to implement this feature
```

This step forces the agent to think through the actual work—what files need to change, what new code is required, what tests need to be written, and what edge cases exist. The plan becomes the basis for the estimate.

### 3. Analyze Historical Data

Here's where the agent's capabilities really shine. It reads through:

- **Git commit history**: Looking at recent commits to understand the codebase velocity
- **Historical story point allocations**: Reading past tickets and their estimates from your issue tracker
- **Lines of code per story point**: Calculating rough metrics from completed work

```
Look at the last 10 completed tickets in PROJ, read their story points,
and analyze the associated git commits to understand lines of code and complexity per point
```

### 4. Generate the Estimate with Justification

Finally, the agent produces an estimate with clear reasoning:

```
Based on your implementation plan and historical analysis, estimate story points for PROJ-456
with detailed justification
```

The agent considers factors like:

- **Lines of code**: How does the expected change size compare to historical tickets?
- **Code complexity**: Is this touching complex algorithms, simple CRUD, or integration points?
- **Testing requirements**: Does this need extensive test coverage?
- **Risk factors**: Are there unknowns or dependencies that add uncertainty?
- **Comparison to similar work**: How does this compare to past tickets the team has completed?

## Why This Works

GenAI agents are well-suited for estimation because they can:

1. **Process large amounts of context**: Read entire ticket histories, codebases, and commit logs
2. **Identify patterns**: Recognize similarities between new work and completed work
3. **Provide consistent reasoning**: Apply the same analysis framework every time
4. **Show their work**: Explain exactly why they arrived at a particular estimate

The estimates I've received have been remarkably reasonable—neither the chronic underestimation nor padding that often plagues human estimates.

## Real-World Results

The proof is in the productivity. My historical baseline was approximately **20 story points per month** within a standard deviation—a consistent measure over time.

After integrating GenAI agents into my workflow—not just for estimation, but for the actual development work—I completed **128 story points in 4 weeks**.

That's a **6x increase** in throughput.

This isn't about the estimates being inflated. The same estimation methodology was applied, the same team reviewed the work, and the same definition of done was met. GenAI agents simply allow you to move faster through the actual implementation while maintaining quality.

## Setting Up Agent-Based Estimation

To enable this workflow, you need:

1. **Issue tracker API access**: Scripts or CLI tools to read tickets and story points
2. **Git repository access**: The agent needs to read commit history
3. **Clear prompting**: Guide the agent through the estimation steps

Example prompt to kick off the full workflow:

```
I need you to estimate story points for PROJ-456. Please:

1. Read the ticket and summarize the requirements
2. Create an implementation plan
3. Look at the last 10 completed tickets, read their story points, and analyze
   the associated commits for lines of code and complexity
4. Provide a story point estimate with detailed justification comparing to historical work
```

## Considerations

A few things to keep in mind:

- **Calibration matters**: The agent needs access to historical data from your specific team to calibrate estimates
- **Review the reasoning**: Always read the justification—it helps you catch misunderstandings
- **Team context**: Story points are team-specific; the agent learns your team's velocity from your data
- **Refinement over time**: The more historical data available, the better the estimates become

## Conclusion

GenAI agents bring a data-driven approach to story point estimation. By analyzing historical patterns, planning implementation details, and comparing to past work, they produce well-reasoned estimates that align with team velocity.

More importantly, these same agents can then help you execute the work at speeds previously unimaginable. When you combine accurate estimation with accelerated delivery, you get predictable planning and dramatically improved throughput.

The 128 story points in 4 weeks wasn't an anomaly—it's the new baseline when GenAI agents are integrated effectively into your development workflow.

## Using Jira for Story Points

If your team uses Jira, you'll need scripts that allow the agent to read and update story points. The Atlassian CLI doesn't provide direct support for story point operations, so I created custom bash scripts that interact with the Jira REST API.

These scripts enable the agent to:

- **Read story points** for one or more tickets to analyze historical estimates
- **Update story points** after the agent completes its estimation analysis

For setup instructions and the scripts themselves, see [Jira Story Points Scripts for GenAI Agents](https://www.nathanfox.net/p/jira-storypoints-scripts-genai-agents).
