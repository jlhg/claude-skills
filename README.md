# Claude Skills

## Available Skills

- **development-preferences** - General development preferences and writing style conventions
- **git-conventions** - Git commit message conventions with Conventional Commits
- **rails-development** - Ruby on Rails development workflow and RSpec testing
- **python-development** - Python packaging and PyPI publishing guidelines

For detailed information about each skill, see the [skills/](skills/) directory.

## Installation

### For Personal Use (All Projects)

Copy skills to your home directory:

```bash
# Copy all skills
cp -r skills/* ~/.claude/skills/

# Or copy specific skills
cp -r skills/development-preferences ~/.claude/skills/
cp -r skills/git-conventions ~/.claude/skills/
```

### For Project-Specific Use

Copy skills to your project directory:

```bash
# Create project skills directory
mkdir -p .claude/skills

# Copy specific skills (e.g., Rails projects)
cp -r skills/rails-development .claude/skills/
cp -r skills/git-conventions .claude/skills/
```

### Verify Installation

```bash
# List personal skills
ls ~/.claude/skills/

# List project skills (if in a project directory)
ls .claude/skills/

# View a specific skill
cat ~/.claude/skills/git-conventions/SKILL.md
```

### Restart Claude Code

After installation, restart Claude Code to load the new skills:

```bash
# If Claude Code is running, restart it
# Skills will be automatically discovered and loaded
```

## How It Works

Claude Code automatically discovers and uses these skills based on your requests:

- **Automatic activation:** Skills are model-invoked - Claude decides when to use them based on the skill description and your request
- **No explicit invocation needed:** Simply work on your tasks naturally
- **Progressive loading:** Claude loads only the necessary information when needed

## Repository Structure

```
claude-skills/
├── skills/
│   ├── development-preferences/
│   ├── git-conventions/
│   ├── rails-development/
│   └── python-development/
└── README.md
```

## Benefits

- **Modular:** Each skill is independent and easy to maintain
- **Efficient:** Claude loads only relevant skills, saving context window
- **Precise triggering:** Skills activate based on task type
- **Shareable:** Commit skills to git and share with your team
- **Extensible:** Easy to add more skills for other languages or frameworks

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
