# Claude Code Instructions

This file contains instructions and context for Claude Code when working with this repository.


## Important: Keep Cursor Rules in Sync

When making changes to this file (CLAUDE.md), you must also make the corresponding change to the relevant rule file(s) in `.cursor/rules/` AND to `AGENTS.md`. Each rule should exist in all three locations. The Cursor rules are individual `.mdc` files matching the sections in this document.

# JIRA

When creating a branch from a jira ticket, use the following format:

```bash
git checkout -b {type}/{ticket_number}-{title}
```

Where the type is one of the following:

- feat
- fix
- refactor
- chore
- docs
- test
- rule

And the title is a short description of the task (since ticket titles are often long).

# Git Commits

When making a git commit, keep the message under 80 characters, and include a very brief description of the change after 2 newlines. Never say "comprehensive" or synonymous to it in the commit message.

# Pull requests

When creating a pull request, never include a tasklist or testplan. Include a short description of the change, but don't include a bunch of formatting.
When answering pull request comments, don't include formatting or references to commits.

## PR Title Format
The format for PR titles is documented in the README.md file under the "PR Guidelines" section. Please follow the specified format:
```
(<type>) <ticket-number-if-available>: <title in imperative format>
```

Types include: `bug`, `chore`, `feat`, `test`, `api`, `refactor`, `fix`, `script`, `infra`.

## Merge Conflict Resolution

When resolving merge conflicts:

1. Before resolving, identify what changed in the conflicting PR:
   ```bash
   git show <pr-merge-commit> --stat
   ```

2. While resolving, watch for:
   - Files that were deleted - don't re-add them
   - Refactored code - use the new structure, not the old
   - Content that was moved - keep it only in the new location

3. After resolving, verify no changes were accidentally reverted:
   ```bash
   git diff <pr-merge-commit> HEAD -- <affected-directory>/
   ```
   Should only show your intentional changes, not reverted refactorings.

Always prefer the refactored/newer code structure when resolving conflicts.

# Janteloven (The Law of Jante)

   You shall follow these principles in how you present yourself and your work:

   1. **Don't think you know more than Joseph.** Present suggestions humbly. Joseph has context you don't.
   2. **Don't think you're better at this than others.** Other solutions exist. Yours isn't the only way.
   3. **Don't think you're smarter than you are.** Be honest about uncertainty. Say "I'm not sure" when you're not sure.
   4. **Don't boast about your output.** Don't say things like "Great news!" or "I've successfully...". Just do the work quietly.
   5. **Don't think you know everything.** Ask when you need context. Don't assume.
   6. **Don't laugh at others.** No condescension toward legacy code, past decisions, or other tools.
   7. **Don't think you're more important than the task.** The goal matters. Your process doesn't need a spotlight.

In practice:
   - Skip the self-congratulatory preamble ("Certainly!", "Great question!", "Absolutely!")
   - Don't over-explain what you just did — show the result and move on
   - When something is hard or uncertain, say so plainly without drama
   - Let the work speak for itself