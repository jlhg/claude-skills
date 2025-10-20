---
name: rails-development
description: Ruby on Rails development conventions and workflows. Use when working with Rails projects including environment setup, RuboCop style enforcement, RSpec testing, and documentation updates.
---

# Rails Development

## Overview

Comprehensive Ruby on Rails development guidelines covering development environment setup, code style, testing with RSpec, and documentation practices.

## Development Environment

### Environment Setup

Access Rails development environment using your project's setup (Docker, local, etc.).

Common tasks to run in the development environment:
- **Rails console**: Access interactive Rails console for debugging and testing
- **Database migrations**: Run pending migrations to update database schema
- **Bundle install**: Install or update Ruby gem dependencies

When running these commands, use the appropriate method for your project (e.g., through Docker, directly on local machine, or via project-specific scripts).

## Code Style

### RuboCop

Follow RuboCop rules defined in:
- Project root: `.rubocop.yml`
- User home: `~/.rubocop.yml`

Format code after changes:

```bash
rubocop <file_or_directory> --auto-correct
```

Or use the short form:

```bash
rubocop <file_or_directory> -a
```

## Testing

### RSpec Workflow

Run RSpec tests using your project's test environment setup.

Test execution patterns:
- **Run all specs**: Execute the entire test suite
- **Run specific spec file**: Test a single file (e.g., `spec/models/user_spec.rb`)
- **Run specific example**: Test a specific line in a spec file (e.g., `spec/models/user_spec.rb:42`)

Ensure tests run in the test environment (`RAILS_ENV=test`) and use `bundle exec rspec` to execute tests through the project's gem dependencies.

### RSpec Best Practices

Follow RSpec best practices documented in [rspec-best-practices.md](references/rspec-best-practices.md).

Key principles:
- Use `describe` with `.` for class methods, `#` for instance methods
- Use `context` for different scenarios
- Define shared helpers/mocks in outer `describe` blocks
- Use `let` for lazy loading, `let!` for eager loading
- Reuse outer `before` blocks in nested contexts
- Use `match(...)` and `hash_including(...)` for validations
- Keep test descriptions clear and concise

## Documentation

After implementing features or changes:

1. Update relevant documentation in the project's documentation directory
2. Keep documentation in sync with code changes
3. Include examples and usage guidelines
4. Update API documentation if endpoints change

## Architecture Updates

When making architectural changes or adding new development conventions:

1. Check if project's `CLAUDE.md` needs updates
2. Document new patterns or conventions
3. Update skill files if necessary
4. Communicate changes to team

## Resources

- [RSpec Best Practices](references/rspec-best-practices.md) - Comprehensive RSpec testing guidelines
