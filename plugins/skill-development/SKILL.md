---
name: Developing Claude Skills
description: Guidelines for authoring effective Claude Skills covering conciseness, progressive disclosure, and discoverable descriptions. Use when creating new skills, refining existing skills, debugging skill discovery issues, optimizing skill descriptions, or when user mentions skill development, SKILL.md, skill authoring, or frontmatter.
---

# Skill Development

## Overview

Guidelines for creating effective Claude Skills that are concise, well-structured, and discoverable. This skill helps maintain consistency across skill development and follows official best practices.

## Core Principles

### Be Concise

Claude is already knowledgeable. Only add context that Claude doesn't have:
- Challenge each piece of information: "Does Claude really need this?"
- Avoid explaining basic concepts (PDFs, libraries, common patterns)
- Keep SKILL.md body under 500 lines

### Set Appropriate Freedom Levels

Match specificity to task fragility:
- **High freedom** (text-based): Multiple valid approaches, context-dependent decisions
- **Medium freedom** (pseudocode/templates): Preferred patterns with acceptable variation
- **Low freedom** (exact scripts): Fragile operations requiring specific sequences

### Use Progressive Disclosure

SKILL.md serves as a table of contents pointing to detailed resources:
- Keep overview in SKILL.md
- Split detailed content into separate reference files
- Reference files one level deep from SKILL.md
- Include table of contents in files longer than 100 lines

## Skill Structure

### YAML Frontmatter

Required fields:
- `name`: Human-readable name using gerund form (e.g., "Processing PDFs") - max 64 characters
- `description`: What the skill does + when to use it + key terms - max 1024 characters

**Description guidelines:**
- Write in third person
- Include specific triggers and contexts
- Use concrete keywords for discovery

Example:
```yaml
---
name: Processing PDFs
description: Extracts text and tables from PDF files, fills forms, merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
```

### Organization Patterns

**Pattern 1: Simple skill**
```
skill-name/
└── SKILL.md
```

**Pattern 2: With references**
```
skill-name/
├── SKILL.md
├── reference.md
└── examples.md
```

**Pattern 3: Domain-specific**
```
skill-name/
├── SKILL.md
└── reference/
    ├── domain-a.md
    ├── domain-b.md
    └── domain-c.md
```

## Writing Guidelines

### Use Consistent Terminology

Choose one term and use it throughout:
- ✓ Always "API endpoint"
- ✗ Mix "API endpoint", "URL", "API route", "path"

### Avoid Time-Sensitive Information

Don't include dates that will become outdated:
- Use "Current method" and "Old patterns" sections
- Document deprecated features in collapsed details blocks

### Provide Templates

For output format requirements:
- Use "ALWAYS use this exact template" for strict requirements
- Use "Here's a sensible default, adapt as needed" for flexible guidance

### Include Examples

For quality-dependent outputs, show input/output pairs:
```markdown
**Example 1:**
Input: [description]
Output:
```
[expected output]
```
```

## Development Workflow

### Iterative Development with Claude

1. **Complete a task without a skill**: Notice what context you repeatedly provide
2. **Identify the reusable pattern**: What information would be useful for similar tasks?
3. **Ask Claude to create a skill**: "Create a skill that captures this pattern"
4. **Review for conciseness**: Remove unnecessary explanations
5. **Test on similar tasks**: Use the skill with a fresh Claude instance
6. **Iterate based on observation**: Refine based on actual usage patterns

### Testing Considerations

Test with all target models:
- **Haiku**: Does it provide enough guidance?
- **Sonnet**: Is it clear and efficient?
- **Opus**: Does it avoid over-explaining?

## Anti-Patterns to Avoid

- ✗ Windows-style paths (`docs\file.md`) - use forward slashes
- ✗ Offering too many options - provide a default with escape hatch
- ✗ Deeply nested references - keep references one level from SKILL.md
- ✗ Assuming packages are installed - be explicit about dependencies
- ✗ Vague descriptions - include specific use cases and keywords

## Comprehensive Reference

For detailed authoring guidelines, examples, and advanced patterns:

**See [references/skill-authoring-best-practices.md](references/skill-authoring-best-practices.md)**

This reference covers:
- Progressive disclosure patterns with visual examples
- Workflows and feedback loops
- Skills with executable code
- Template and conditional workflow patterns
- Evaluation and iteration strategies
- Technical requirements and token budgets
- Complete checklist for effective skills
