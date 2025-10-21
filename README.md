# Claude Skills

## Available Plugins and Skills

This repository contains 2 plugins with 5 skills total.

### development-skills

Development skills for Git conventions and language-specific best practices

- **development-preferences** - General development preferences and writing style conventions
- **git-conventions** - Git commit message conventions with Conventional Commits
- **python-development** - Python packaging and PyPI publishing guidelines
- **rails-development** - Ruby on Rails development workflow and RSpec testing

### agent-skills

Skills for developing and authoring Claude Skills following best practices

- **skill-development** - Guidelines and best practices for authoring effective Claude Skills

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
cp -r plugins/* ~/.claude/skills/

# For project-specific use
mkdir -p .claude/skills
cp -r plugins/rails-development .claude/skills/
```

### Restart Claude Code

After installation, restart Claude Code to load the new skills.

## License

This project is licensed under the [MIT License](LICENSE).
