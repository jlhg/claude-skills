---
name: git-conventions
description: Git commit message conventions following Conventional Commits. **Provides commit messages only - does not execute git commands unless explicitly requested.** Use for commit message generation and GPG signing guidance.
---

# Git Conventions

## Overview

Follow Conventional Commits specification for all commit messages with GPG signing requirements.

## Commit Message Guidelines

### Format

Generate commit messages that follow Conventional Commits specification:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Requirements

- Be concise and clear
- Match the current project's style
- Write in English
- Use imperative mood in subject line (e.g., "add" not "added" or "adds")
- Keep subject line under 72 characters
- Separate subject from body with a blank line
- Wrap body at 72 characters

### Workflow

**IMPORTANT: Only provide the commit message. Do NOT execute git commands unless explicitly requested by the user.**

Default behavior:
1. Analyze the changes being made
2. Choose the appropriate commit type from references
3. Write a clear, descriptive commit message
4. Present the commit message to the user
5. Remind user to sign with GPG key

**Do NOT:**
- Run `git add` unless user explicitly asks
- Run `git commit` unless user explicitly asks
- Execute any git commands automatically

**Only execute git commands when user explicitly requests it with phrases like:**
- "please commit this for me"
- "can you run git add and commit"
- "commit these changes with this message"

Example output format:
```
Here's your commit message:

feat(auth): add JWT token refresh mechanism

Implement automatic token refresh before expiration to improve
user experience. Tokens are refreshed 5 minutes before expiry.

Closes #123

You can commit with:
git commit -S -m "feat(auth): add JWT token refresh mechanism"
```

## GPG Signing

All commits must be signed with GPG key. Remind users to use:

```bash
git commit -S -m "commit message"
```

Or configure automatic signing:

```bash
git config --global commit.gpgsign true
```

## Resources

See [commit-types.md](references/commit-types.md) for complete Conventional Commits type reference.
