---
name: rails-development
description: Ruby on Rails development conventions and workflows. Use when working with Rails projects including Docker setup, RuboCop style enforcement, RSpec testing, and documentation updates.
---

# Rails Development

## Overview

Comprehensive Ruby on Rails development guidelines covering Docker environment, code style, testing with RSpec, and documentation practices.

## Development Environment

### Docker Setup

Access Rails development environment using:

```bash
docker compose -f compose.local.yaml exec -it web <command>
```

Common commands:
- Rails console: `docker compose -f compose.local.yaml exec -it web rails console`
- Database migrations: `docker compose -f compose.local.yaml exec -it web rails db:migrate`
- Bundle install: `docker compose -f compose.local.yaml exec -it web bundle install`

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

Run tests using Docker:

```bash
docker compose -f compose.local.yaml run --rm -e RAILS_ENV=test web bundle exec rspec <spec_file>
```

Examples:
```bash
# Run all specs
docker compose -f compose.local.yaml run --rm -e RAILS_ENV=test web bundle exec rspec

# Run specific spec file
docker compose -f compose.local.yaml run --rm -e RAILS_ENV=test web bundle exec rspec spec/models/user_spec.rb

# Run specific example
docker compose -f compose.local.yaml run --rm -e RAILS_ENV=test web bundle exec rspec spec/models/user_spec.rb:42
```

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

1. Update relevant documentation in `doc/src/`
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
