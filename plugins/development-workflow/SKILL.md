---
name: Following Development Workflows
description: Provides development workflow guidelines including incremental progress, test-driven development, planning stages, and quality standards. Use when implementing features, refactoring code, fixing bugs, writing tests, making commits, or when user mentions development, coding, implementation, planning, architecture, build, or quality.
---

# Development Workflow

## Overview

Follow systematic development practices that prioritize incremental progress, testability, and code quality. This skill provides guidelines for planning, implementing, and maintaining high-quality code across all development activities.

## Core Principles

### Philosophy

- **Incremental progress over big bangs** - Make small changes that compile and pass tests
- **Learning from existing code** - Study patterns and plan before implementing
- **Pragmatic over dogmatic** - Adapt to project reality
- **Clear intent over clever code** - Choose boring, obvious solutions

### Simplicity

- Single responsibility per function/class
- Avoid premature abstractions
- No clever tricks - be obvious
- If explanation is needed, it's too complex

## Process Guidelines

### Planning Complex Work

For complex tasks, break work into 3-5 stages in `IMPLEMENTATION_PLAN.md`:

```markdown
## Stage N: [Name]
**Goal**: [Specific deliverable]
**Success Criteria**: [Testable outcomes]
**Tests**: [Specific test cases]
**Status**: [Not Started|In Progress|Complete]
```

- Update status as you progress
- Remove file when complete

### Implementation Flow

1. **Understand** - Study existing patterns in codebase
2. **Test** - Write test first (red)
3. **Implement** - Minimal code to pass (green)
4. **Refactor** - Clean up with tests passing
5. **Commit** - With clear message linking to plan

### When Stuck (After 3 Attempts)

**CRITICAL**: Maximum 3 attempts per issue, then STOP.

1. Document what failed and why
2. Research 2-3 alternative approaches
3. Question fundamentals - is this the right abstraction?
4. Try a completely different angle

## Technical Standards

### Architecture

- Composition over inheritance
- Interfaces over singletons
- Explicit over implicit
- Test-driven when possible

### Every Commit Must

- Compile successfully
- Pass all existing tests
- Include tests for new functionality
- Follow project formatting/linting

### Error Handling

- Fail fast with descriptive messages
- Include context for debugging
- Handle errors at appropriate level
- Never silently swallow exceptions

## Decision Framework

When multiple valid approaches exist, prioritize:

1. **Testability** - Can I easily test this?
2. **Readability** - Will someone understand this in 6 months?
3. **Consistency** - Does this match project patterns?
4. **Simplicity** - Is this the simplest solution that works?
5. **Reversibility** - How hard to change later?

## Quality Gates

### Definition of Done

- [ ] Tests written and passing
- [ ] Code follows project conventions
- [ ] No linter/formatter warnings
- [ ] Commit messages are clear
- [ ] Implementation matches plan
- [ ] No TODOs without issue numbers

### Test Guidelines

- Test behavior, not implementation
- One assertion per test when possible
- Clear test names describing scenario
- Use existing test utilities/helpers
- Tests should be deterministic

## Important Reminders

**NEVER**:
- Use `--no-verify` to bypass commit hooks
- Disable tests instead of fixing them
- Commit code that doesn't compile
- Make assumptions - verify with existing code

**ALWAYS**:
- Commit working code incrementally
- Update plan documentation as you go
- Learn from existing implementations
- Stop after 3 failed attempts and reassess

## Detailed Reference

For comprehensive guidelines covering all aspects of development workflow:

**See [references/development-guidelines.md](references/development-guidelines.md)**

This reference includes:
- Extended philosophy and beliefs
- Detailed process workflows
- Project integration strategies
- Complete tooling guidelines
- Expanded quality standards
