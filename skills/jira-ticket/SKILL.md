---
name: jira-ticket
description: Start working on a Jira ticket - fetches context and marks it in progress. Use when starting work on a ticket like "work on KEY-123" or "start KEY-456".
argument-hint: <TICKET-KEY> [additional context]
allowed-tools: Bash(acli jira:*)
disable-model-invocation: false
---

# Work on Jira Ticket

You are starting work on a Jira ticket.

## Ticket Information

**Ticket Key**: $ARGUMENTS

## Step 1: Gather Ticket Context

Use the **jira-context** agent to fetch full details about the ticket:

```
Launch jira-context agent with the ticket key from arguments
```

## Step 2: Mark Ticket In Progress

After gathering context, transition the ticket to "In Progress":

```bash
acli jira workitem transition --key "<TICKET_KEY>" --status "In Progress"
```

## Step 3: Assign Ticket to Me

```bash
acli jira workitem assign --key "<TICKET_KEY>" --assignee "@me"
```

## Step 4: Present Work Summary

After gathering context and transitioning, present:

1. **Task Summary**: Brief overview of what needs to be done
2. **Acceptance Criteria**: What defines "done"
3. **Additional Context**: Any notes from the user
4. **Suggested Approach**: Your recommended approach to implement this

## Step 5: Begin Implementation

Ask if the user wants to:
- Start implementing immediately
- Explore the codebase first
- Discuss the approach

## When Work is Complete

When the user indicates they're done or wants review, use the **code-reviewer** agent to review the changes against the task requirements before committing.

Remember: Keep the task context in mind throughout the session for accurate code review at the end.
