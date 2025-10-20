# Claude Skills

## Available Skills

- **development-preferences** - General development preferences and writing style conventions
- **git-conventions** - Git commit message conventions with Conventional Commits
- **rails-development** - Ruby on Rails development workflow and RSpec testing
- **python-development** - Python packaging and PyPI publishing guidelines

For detailed information about each skill, see the individual skill directories.

## Installation

### Via Plugin Marketplace (Recommended)

Add this repository to your Claude Code plugin marketplace:

```bash
/plugin marketplace add jlhg/claude-skills
```

Then browse and install skills using the `/plugin` menu.

### Manual Installation

```bash
# For personal use (all projects)
cp -r development-preferences git-conventions python-development rails-development ~/.claude/skills/

# For project-specific use
mkdir -p .claude/skills
cp -r rails-development .claude/skills/
```

### Restart Claude Code

After installation, restart Claude Code to load the new skills:

```bash
# If Claude Code is running, restart it
# Skills will be automatically discovered and loaded
```

## License

This project is licensed under the [MIT License](LICENSE).
