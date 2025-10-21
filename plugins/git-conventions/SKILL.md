---
name: Following Git Conventions
description: Provides Git commit message conventions following Conventional Commits specification.
---

# Git Conventions

## Overview

Follow Conventional Commits specification for all commit messages. Provides GPG signing guidance based on the user's git configuration.

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
4. Present the commit message to the user (without command examples)

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
feat(auth): add JWT token refresh mechanism

Implement automatic token refresh before expiration to improve
user experience. Tokens are refreshed 5 minutes before expiry.

Closes #123
```

## GPG Signing

**Only relevant when Claude executes git commands.**

When the user explicitly requests Claude to execute git commit commands:

1. Check if GPG signing is enabled:
   ```bash
   git config --get commit.gpgsign
   ```

2. Use the appropriate command:
   - If output is `true`: Use `git commit -S -m "message"`
   - Otherwise: Use `git commit -m "message"`

## Resources

See [commit-types.md](references/commit-types.md) for complete Conventional Commits type reference.
