# Claude Skills Project

Custom Claude Code skills for development workflows.

## 🚨 CRITICAL: File Location

**ALWAYS edit files in THIS project, NEVER in `~/.claude/`!**

## Progressive Disclosure

Skills use a three-level loading system:

1. **Metadata (description)** - Always in context (~50 words)
   - Determines WHEN to load
   - Keep concise: list only common file patterns
2. **SKILL.md body** - Loaded when triggered (<5k words)
   - Focus on HOW to use
   - NO trigger conditions or "Auto-Loading" sections
3. **Bundled resources** - Loaded as needed
   - `references/` for detailed docs

## Guidelines

### ✅ DO

- Keep metadata under 60 words
- Focus body on workflows
- Remove trigger conditions from body
- Use imperative form (not second person)

### ❌ DON'T

- Edit files in `~/.claude/`
- Repeat trigger conditions in body
- Include "Environment Detection" in body
- Add "Important: Always apply..." in body
- Duplicate metadata in body
