---
name: fix-pr
description: Checks the PR for the current branch on GitHub and fixes any issues — unaddressed review comments and failing CI checks.
---

# Fix PR

Fix all outstanding issues on the pull request for the currently checked out branch.

## Step 0: Identify the PR

```bash
gh pr view --json number,url,title,state --jq '.number'
```

If no PR exists for the current branch, stop and inform the user.

## Step 1: Address review comments

### Fetch unresolved comments

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --jq '.[] | select(.in_reply_to_id == null) | {id, path, line, body, user: .user.login}'
```

Also check for top-level review comments (non-inline):

```bash
gh pr view {pr_number} --json reviews --jq '.reviews[] | select(.state != "APPROVED") | {author: .author.login, body: .body, state: .state}'
```

### For each unresolved comment

1. **Read the relevant code** in the file and around the referenced line(s).
2. **Evaluate** whether the comment is actionable and correct:
   - If the suggestion is valid → make the fix, then reply concisely: "Fixed" / "Done" / a one-liner explaining what changed.
   - If the suggestion is wrong or not applicable → reply explaining why you disagree, referencing the code or context. Be respectful but direct.
   - If the comment is a question or FYI with no action needed → reply answering the question or acknowledging.
3. **Do not** make changes beyond what the comment asks for. Stay scoped.

### Reply to comments via CLI

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments/{comment_id}/replies -f body="Fixed"
```

### Rules for addressing comments

- Never ignore a comment. Every comment gets a reply.
- Do not reply with only "Acknowledged" or "Noted" if an actual code change was requested — either fix it or explain why not.
- If multiple comments ask for the same change, fix it once and reference the fix in all replies.
- If a comment is outdated (the code it references has already changed), say so: "This was already addressed in [commit/change]."
- Batch related file edits together — don't make 5 separate commits for 5 comments on the same file.
- Commit with a clear message: `fix: address PR review feedback`

## Step 2: Fix failing checks

### Check status

```bash
gh pr checks {pr_number} --json name,state,link --jq '.[] | select(.state != "SUCCESS" and .state != "SKIPPED")'
```

### For each failing check

1. **Get the logs:**
   ```bash
   gh run view {run_id} --log-failed
   ```
   If `--log-failed` produces no output, try `--log` and search for errors.

2. **Diagnose** the failure — common categories:
   - Lint / format errors → run the project's linter/formatter and commit the fix.
   - Type errors → fix the type issue in the relevant file.
   - Test failures → read the failing test, understand what it expects, fix the code (not the test) unless the test itself is wrong given the PR's intent.
   - Build failures → check for missing imports, syntax errors, or dependency issues.

3. **Fix and push** the changes.

4. **Wait for checks to re-run:**
   ```bash
   gh pr checks {pr_number} --watch --fail-fast
   ```

5. If checks fail again, repeat. After 3 failed attempts on the same check, stop and inform the user — something likely needs human judgment.

### Rules for fixing checks

- Do not skip or ignore failing checks.
- Do not modify CI configuration files (e.g., `.github/workflows/`) to make checks pass unless the PR is specifically about CI changes.
- Do not disable or delete tests to make them pass.
- If a check failure is clearly unrelated to the PR (flaky test, infrastructure issue), inform the user rather than trying to fix it.

## Step 3: Request re-review

After addressing review comments and/or fixing checks, request re-review from all reviewers who left comments or requested changes:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews --jq '[.[] | select(.state == "CHANGES_REQUESTED" or .state == "COMMENTED") | .user.login] | unique | .[]'
```

Then request re-review from each:

```bash
gh pr edit {pr_number} --add-reviewer {reviewer_login}
```

## Step 4: Final verification

After all comments are addressed and checks are green:

1. Confirm no new comments have appeared while you were working:
   ```bash
   gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --jq 'length'
   ```
2. Confirm all checks pass:
   ```bash
   gh pr checks {pr_number}
   ```
3. Report a summary to the user: how many comments were addressed, what was fixed, current check status, and who was requested for re-review.

## Step 5: Loop until approved

After completing steps 0–4, use the `/loop` skill to re-run `/fix-pr` every 5 minutes. This will keep checking for new review comments and failing checks until the PR is approved.

Before each iteration, check if the PR has been approved:

```bash
gh pr view {pr_number} --json reviews --jq '.reviews[] | select(.state == "APPROVED") | .user.login'
```

If the PR has at least one approval and no outstanding `CHANGES_REQUESTED` reviews, stop the loop and inform the user that the PR is approved.