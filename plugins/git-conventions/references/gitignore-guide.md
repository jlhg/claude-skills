# .gitignore Complete Guide

## Purpose

`.gitignore` specifies intentionally untracked files that Git should ignore. Properly configured ignore files help:
- Keep repositories clean and focused
- Prevent sensitive data from being committed
- Reduce repository size by excluding build artifacts
- Avoid conflicts across different development environments

## What to Ignore

Typically exclude:
- Build artifacts and compiled code
- Dependency directories (e.g., `node_modules/`, `vendor/`)
- Log files (`*.log`)
- IDE/editor-specific files
- OS-specific files (e.g., `.DS_Store`, `Thumbs.db`)
- Temporary files
- Environment configuration files with secrets (`.env`)

## Format and Syntax

1. **Comments and blank lines**:
   - Lines starting with `#` are comments
   - Blank lines are ignored

2. **Glob patterns**:
   - `*` matches anything except a slash
   - `?` matches any single character except `/`
   - `**` matches any number of subdirectories
   - `[a-zA-Z]` matches one character in a range

3. **Negation**:
   - Prefix `!` negates a pattern (re-includes files)
   - Use `\!` to match files starting with `!`

4. **Slash behavior**:
   - `/` at start/middle makes patterns relative to `.gitignore` location
   - `/` at end matches only directories
   - No `/` means pattern matches at any level

## Pattern Examples

```gitignore
# Ignore all .log files
*.log

# Ignore all files in logs/ directory
logs/

# Ignore myFile1 only in root directory
/myFile1

# Ignore all files in myDir except myFile.txt
myDir/*
!myDir/myFile.txt

# Ignore all .pdf files in docs/ and subdirectories
docs/**/*.pdf
```

## Security Best Practices

**IMPORTANT**: Do NOT rely solely on `.gitignore` for sensitive data protection.

Additional measures:
- Use tools like `git-secrets` or `gitleaks` to scan for secrets
- Consider encrypting sensitive files with `git-crypt`
- Store secrets outside the repository when possible
- Use environment variables for sensitive configuration
- Implement pre-commit hooks to prevent accidental commits

## File Locations and Precedence

### Three Types of Ignore Files

1. **`.gitignore`**:
   - Project-specific, tracked in repository
   - Shared with all collaborators
   - Can be placed in any directory (affects that directory and subdirectories)

2. **`~/.gitignore_global`**:
   - User-specific, applies to all repositories
   - Not tracked by Git
   - Good for editor/IDE files

3. **`.git/info/exclude`**:
   - Repository-specific, not tracked
   - Local to your machine only
   - Not shared with collaborators

### Precedence Order (highest to lowest)

1. Command-line patterns (e.g., `git clean -e`)
2. `.gitignore` files (inner directories override outer)
3. `.git/info/exclude`
4. `core.excludesFile` configuration

## Common Tasks

### Check which files are ignored

```bash
# Check specific file
git check-ignore -v <filename>

# Check with non-matching output
git check-ignore -v -n <filename>

# Check all files in directory
find . -not -path './.git/*' | git check-ignore --stdin -v
```

### Stop tracking a previously tracked file

**Important**: `.gitignore` only affects untracked files. To ignore a tracked file:

```bash
# Remove from Git but keep local file
git rm --cached <filename>

# Commit the change
git commit -m "chore: stop tracking <filename>"
```

⚠️ **Warning**: Some sources suggest `git rm --cached` may remove the file from your working directory in certain scenarios. Always backup important files first.

### Handle exceptions (negation patterns)

When using negation, remember Git doesn't scan ignored directories:

```gitignore
# This WON'T work (directory is ignored, so file inside is never checked)
myDir/
!myDir/myFile.txt

# This WORKS (ignore files in directory, but still check for exceptions)
myDir/*
!myDir/myFile.txt
```

## Tracked vs Untracked vs Ignored Files

- **Tracked files**: Already added to Git, changes are monitored
- **Untracked files**: Exist in working directory but not in Git
- **Ignored files**: Untracked files explicitly excluded by `.gitignore`

**Key point**: Adding a tracked file to `.gitignore` does NOT stop Git from tracking it. You must first untrack it with `git rm --cached`.

## Templates and Resources

### GitHub Templates

- [GitHub's gitignore repository](https://github.com/github/gitignore) - Official template collection
- GitHub offers templates when creating new repositories

### Online Generators

- [gitignore.io](https://www.toptal.com/developers/gitignore) - Generate custom .gitignore files
- GitLab provides pre-populated projects with .gitignore

### Template Examples

For LaTeX projects:
```gitignore
## Core latex/pdflatex auxiliary files
*.aux
*.lof
*.log
*.lot
*.out
*.toc

## Intermediate documents
*.dvi
```

For Python projects:
```gitignore
# Byte-compiled files
*.pyc
__pycache__/

# Virtual environments
venv/
env/
.venv/

# Distribution/packaging
dist/
build/
*.egg-info/
```

For Node.js projects:
```gitignore
# Dependencies
node_modules/

# Build output
dist/
build/

# Environment files
.env
.env.local

# Logs
*.log
npm-debug.log*
```

## Troubleshooting

### File still tracked after adding to .gitignore

The file was tracked before adding to `.gitignore`. Solution:
```bash
git rm --cached <filename>
git commit -m "chore: untrack <filename>"
```

### Gitignore not working

Common causes:
1. File is already tracked (see above)
2. Pattern syntax error
3. Cached index - try: `git rm -r --cached . && git add .`

### Debug which rule is ignoring a file

```bash
git check-ignore -v <filename>
# Output shows: .gitignore:2:*.log    myfile.log
#                ^file    ^line ^pattern  ^filename
```

## Best Practices

1. **Commit .gitignore early**: Add it in your first commit
2. **Be explicit**: Prefer specific patterns over wildcards when possible
3. **Document complex patterns**: Add comments explaining non-obvious rules
4. **Don't ignore .gitignore**: Keep it tracked (unless you have specific reasons)
5. **Use templates**: Start with language-specific templates
6. **Test before committing**: Use `git check-ignore` to verify patterns
7. **Keep it organized**: Group related patterns with comments
8. **Review regularly**: Update as your project evolves

## Empty Directories

Git doesn't track empty directories. To keep an empty directory:

```bash
# Create .gitkeep file
touch empty_directory/.gitkeep
git add empty_directory/.gitkeep
```

Alternative names: `.keep` or empty `.gitignore` with `!.gitignore`

## References

For more detailed information, see:
- [github-gitignore.md](github-gitignore.md) - GitHub's template collection
- [the_ultimate_gitignore_guide.md](the_ultimate_gitignore_guide.md) - Comprehensive Q&A guide
