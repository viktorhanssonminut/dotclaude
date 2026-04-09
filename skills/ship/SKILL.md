---
name: ship
description: Review, commit, push, and create draft PR - full shipping flow. Use when ready to ship changes.
argument-hint: [commit message]
allowed-tools: Bash(git:*), Bash(gh pr:*)
---

# Ship Changes

Review, commit, push, and create a draft PR in one flow.

## Commit Message

$ARGUMENTS

## Step 1: Check Current State

```bash
# Check for uncommitted changes
git status

# See what will be committed
git diff HEAD

# Get current branch
git branch --show-current
```

If no changes to commit, inform the user and exit.

## Step 2: Run Code Review

Before committing, invoke the **/review** skill with no arguments (working tree mode). This will:
- Review uncommitted changes using the full review team (bugs, security, conventions, requirements)
- Automatically detect and fetch Jira ticket context from the branch name
- Present findings grouped by reviewer

## Step 3: Handle Review Results

If issues are found, present the review output and ask the user:

```
Would you like to:
1. Fix issues before shipping
2. Ship anyway (for minor issues)
3. Cancel
```

Wait for user decision before proceeding.

## Step 4: Commit, Push, and Create Draft PR

Once approved (or no critical issues):

```bash
# Stage all changes
git add -A

# Commit with message (use provided message or generate one)
git commit -m "<commit message>"

# Push to remote (set upstream if needed)
git push -u origin $(git branch --show-current)

# Create draft PR and capture the full URL
PR_URL=$(gh pr create --draft --fill)
echo "$PR_URL"
```

After the PR is created, always display the full PR URL to the user.

If no commit message provided in arguments, generate one based on the changes.

## Quick Ship Mode

If the user says "ship it" or "just ship", skip the detailed review and proceed directly with commit, push, and draft PR. Only block on critical security issues.
