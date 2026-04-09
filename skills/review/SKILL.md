---
name: review
description: Review code changes — accepts a PR link/number, a commit SHA, or nothing (reviews working tree). Fetches Jira context automatically if a ticket is referenced.
argument-hint: [PR-URL-OR-NUMBER | COMMIT-SHA | blank]
---

# Code Review

Review code changes using a team of specialized agents.

## Input

$ARGUMENTS

## Step 1: Determine Review Mode

Based on the input, determine which mode to use:

- **PR mode**: Input looks like a PR URL, PR number, or `pr` keyword → fetch the PR diff
- **Commit mode**: Input looks like a commit SHA (hex string, 7+ chars) → review that commit
- **Working tree mode**: No input or empty → review uncommitted changes (`git diff HEAD`)

### PR Mode
```bash
gh pr view $ARGUMENTS --json title,body,author,baseRefName,headRefName,url,number
gh pr diff $ARGUMENTS
```
Save the PR number, title, base branch, and diff.

### Commit Mode
```bash
git show $ARGUMENTS --stat
git show $ARGUMENTS
git log -1 --format="%s%n%b" $ARGUMENTS
```
Save the commit message and diff.

### Working Tree Mode
```bash
git diff HEAD
git status
git branch --show-current
git log -1 --format="%s" HEAD
```
Save the diff of uncommitted changes. If there is no diff, tell the user there's nothing to review.

## Step 2: Extract Ticket References

Look for Jira ticket patterns (e.g. `KEY-123`, `[KEY-123]`, `(KEY-123)`) in:
- PR title and body (if PR mode)
- Commit message (if commit mode)
- Current branch name (all modes)

```bash
git branch --show-current
```

## Step 3: Gather Context

If a Jira ticket was found, use the **jira-context** agent to fetch the ticket details and understand the requirements.

Also read the project's CLAUDE.md and any relevant configuration files (.eslintrc, tsconfig, etc.) to understand the codebase conventions.

## Step 4: Spawn Review Team

Create an agent team with **four specialized reviewers** and yourself as team lead. Each reviewer gets the diff, the ticket context (if available), and a focused mandate.

**IMPORTANT:** Each teammate must ONLY report findings within their specific expertise. They must not overlap with other reviewers' domains.

### Teammate 1: Bug Hunter
**Focus:** Finding bugs, logic errors, race conditions, edge cases, and incorrect behavior.
**Prompt:**
> You are reviewing code changes: "{title or summary}".
> Your ONLY job is to find bugs. Look for:
> - Logic errors and off-by-one mistakes
> - Race conditions and concurrency issues
> - Null/undefined handling gaps
> - Edge cases that would cause crashes or incorrect behavior
> - Incorrect API usage or wrong assumptions about data shapes
> - Missing error handling that would cause silent failures
>
> Do NOT comment on style, naming, security, or whether it matches requirements. Only bugs.
>
> For each finding, report: file path, line number(s), what the bug is, and what would happen if it's not fixed.
> If you find nothing, say so -- don't manufacture issues.

### Teammate 2: Requirements Validator
**Focus:** Verifying the implementation matches what the Jira ticket asks for.
**Prompt:**
> You are reviewing code changes: "{title or summary}".
> The Jira ticket says: {ticket context summary}
>
> Your ONLY job is to check whether the changes correctly implement what the ticket requires. Look for:
> - Missing acceptance criteria that aren't implemented
> - Implemented behavior that contradicts the requirements
> - Edge cases mentioned in the ticket that aren't handled
> - Scope creep -- changes unrelated to the ticket
>
> Do NOT comment on code quality, security, or style. Only requirements alignment.
>
> For each finding, report: what requirement is affected, what's missing or wrong, and which file(s) are involved.
> If the changes fully satisfy the ticket, say so.

**Note:** If no Jira ticket was found, skip this reviewer entirely and note that no ticket context was available.

### Teammate 3: Security Auditor
**Focus:** Finding security vulnerabilities and unsafe patterns.
**Prompt:**
> You are reviewing code changes: "{title or summary}".
> Your ONLY job is to find security issues. Look for:
> - Injection vulnerabilities (SQL, XSS, command injection, template injection)
> - Authentication and authorization gaps
> - Sensitive data exposure (logging secrets, hardcoded credentials, PII leaks)
> - Insecure deserialization or unsafe parsing
> - Missing input validation at system boundaries
> - CSRF, SSRF, or open redirect risks
> - Insecure cryptographic usage
>
> Do NOT comment on bugs, style, or requirements. Only security.
>
> For each finding, report: severity (critical/high/medium/low), file path, line number(s), the vulnerability, and suggested fix.
> If you find nothing, say so -- don't manufacture issues.

### Teammate 4: Convention Checker
**Focus:** Ensuring the code follows the project's established patterns and guidelines.
**Prompt:**
> You are reviewing code changes: "{title or summary}".
> Your ONLY job is to check that the code follows this project's conventions and guidelines.
>
> Read the project's CLAUDE.md, AGENTS.md, and any linting/formatting configs to understand the rules.
> Then check for:
> - Violations of documented coding standards
> - Deviations from established patterns in the codebase (naming, file structure, imports)
> - Missing or incorrectly formatted types/interfaces (if TypeScript)
> - Inconsistent error handling patterns vs the rest of the codebase
> - PR title/branch naming convention violations
>
> Do NOT comment on bugs, security, or requirements. Only conventions.
>
> For each finding, report: the convention being violated, file path, line number(s), and what it should look like instead.
> If everything follows conventions, say so.

## Step 5: Collect and Present Findings

Once all teammates have reported back, compile their findings into a single numbered list. **Output must be valid GitHub-flavored markdown** — use proper headings (`#`, `##`, `###`), fenced code blocks for any code snippets, and backticks for inline file paths / identifiers. Do not use `===` or `---` as section separators (use headings instead). Do not wrap the whole report in a code fence.

Use this structure exactly:

````markdown
# Code Review: <Title or Summary>

**Mode:** <PR #123 | Commit `abc1234` | Working Tree>
**Ticket:** <ticket number> — <brief summary> (or _No ticket found_)

## Bug Hunter

1. **Critical** — `src/api/users.ts:45`
   - **Bug:** Array index not bounds-checked, will crash on empty input.
   - **Impact:** Runtime error in production.

## Requirements Validator

2. **Gap** — `src/components/Form.tsx`
   - **Requirement:** "User should see validation errors inline"
   - **Status:** Validation errors only shown as toast, not inline.

## Security Auditor

3. **High** — `src/api/search.ts:34`
   - **Vulnerability:** User input interpolated directly into SQL query.
   - **Fix:** Use a parameterized query.

## Convention Checker

4. **Style** — `src/services/OrderService.ts:1`
   - **Convention:** Service files should use kebab-case naming.
   - **Should be:** `order-service.ts`

## Summary

- **Bugs:** N (breakdown by severity)
- **Requirements gaps:** N
- **Security issues:** N (breakdown by severity)
- **Convention violations:** N

**Overall:** <Ready to ship | Needs changes | Major rework needed>
````

Rules for the output:
- If a reviewer has no findings, still include their `##` section with a single line like `_No findings._` — never omit sections silently (except the Requirements Validator when no ticket was found; in that case include the section with `_Skipped — no ticket context available._`).
- Always wrap file paths, identifiers, env vars, and line refs in backticks.
- Use fenced code blocks (```lang) for any multi-line code suggestions.
- Never emit bare `===` / `---` separators — they render inconsistently and break markdown structure.

## Step 6: Post Comments (PR Mode Only)

If reviewing a PR, ask the user:
"Which comments would you like me to post to the PR? Reply with the numbers (e.g., '1, 3, 4' or 'all' or 'none')"

When the user specifies which comments to post, post them as inline review comments on the relevant diff lines using the GitHub API. **Never** post comments to the main PR conversation thread. Always submit the review as "REQUEST_CHANGES".
When leaving a review using the review skill/mode, only submit the queued inline comments. Do not add a top-level review summary comment.


**Every comment body must be prefixed with the robot emoji.** For example: `"Bug: Array index not bounds-checked..."`

Use the GitHub API to create a review with inline comments:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews \
  --method POST \
  --input review.json
```

Where `review.json` contains:
```json
{
  "event": "REQUEST_CHANGES",
  "body": "Summary of review findings",
  "comments": [
    {
      "path": "src/api/users.ts",
      "line": 45,
      "body": "Bug: Array index not bounds-checked..."
    }
  ]
}
```

After posting, clean up the temporary JSON file.

For commit or working tree mode, just present the findings -- no PR comments to post.
