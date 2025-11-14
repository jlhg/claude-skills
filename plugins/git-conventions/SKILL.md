---
name: Following Git Conventions
description: Generates Conventional Commits specification messages and provides .gitignore configuration guidance. Use when writing commit messages, analyzing git diffs, creating/updating .gitignore files, or when user mentions git, commits, commit conventions, conventional commits, gitignore, or asks to commit changes.
---

# Git Conventions

## Overview

Follow Git best practices including:
- Conventional Commits specification for all commit messages
- GPG signing guidance based on the user's git configuration
- .gitignore configuration and best practices

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
- Add blank line before bullet lists
- Align continuation lines in bullet lists with 2-space indentation

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

Changes:

- Add TokenRefreshService with automatic refresh logic
- Update AuthInterceptor to handle token expiration by checking
  expiry timestamp and triggering refresh 5 minutes before
- Add retry mechanism for failed API calls due to expired tokens

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

## .gitignore Best Practices

### Common Patterns

Typically exclude:
- `*.log` - Log files
- `node_modules/`, `vendor/` - Dependency directories
- `.env` - Environment files with secrets
- `.DS_Store`, `Thumbs.db` - OS-specific files
- `*.pyc`, `__pycache__/` - Python compiled files
- `dist/`, `build/` - Build artifacts

### Key Commands

```bash
git check-ignore -v <filename>  # Check if file is ignored
git rm --cached <filename>      # Stop tracking file
```

### Security Warning

Do NOT rely solely on `.gitignore` for sensitive data protection. Use tools like `git-secrets` or `gitleaks`.

### Detailed References

**Complete guide**: See [gitignore-guide.md](references/gitignore-guide.md) for format, syntax, precedence rules, and workflows

**Templates**: See [github-gitignore.md](references/github-gitignore.md) for GitHub's template collection

**Comprehensive Q&A**: See [the_ultimate_gitignore_guide.md](references/the_ultimate_gitignore_guide.md) for detailed explanations

## Resources

- [commit-types-cheatsheet.md](references/commit-types-cheatsheet.md) - Detailed commit types cheatsheet with usage guidelines
- [gitignore-guide.md](references/gitignore-guide.md) - Complete .gitignore guide with format, syntax, and best practices
- [github-gitignore.md](references/github-gitignore.md) - GitHub's .gitignore template collection and contribution guidelines
- [the_ultimate_gitignore_guide.md](references/the_ultimate_gitignore_guide.md) - Comprehensive .gitignore guide in Q&A format
