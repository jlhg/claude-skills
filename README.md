# Claude Skills

## Available Plugins and Skills

This repository contains 3 plugins with 8 skills total.

### development-skills

Development skills for Git conventions and language-specific best practices

- **git-conventions** - Git commit message conventions with Conventional Commits
- **python-development** - Python packaging and PyPI publishing guidelines
- **ruby-development** - Ruby and Rails development with RuboCop style guide and RSpec testing

### backend-skills

Backend architecture, deployment, performance optimization, and security best practices

- **backend-architecture** - Backend system architecture design with Redis/Valkey and configuration management
- **deployment-practices** - Zero-downtime deployment, Cloudflare Tunnel, and database migration strategies
- **performance-optimization** - Memory management, jemalloc configuration, and performance profiling
- **web-security** - API rate limiting, authentication architecture, and security best practices

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
cp -r plugins/ruby-development .claude/skills/
```

### Restart Claude Code

After installation, restart Claude Code to load the new skills.

## License

This project is licensed under the [MIT License](LICENSE).
