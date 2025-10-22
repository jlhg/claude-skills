---
name: Ruby Development
description: Provides Ruby and Ruby on Rails development conventions including RuboCop formatting, Ruby style guide, RSpec testing, Rails best practices, CurrentAttributes usage, input normalization, frozen string literals, and HTTP client (Faraday) integration. Use when developing Ruby or Rails applications, writing tests, formatting code, handling user input, optimizing string memory, making HTTP requests, or when user mentions Ruby, Rails, RSpec, RuboCop, Active Record, CurrentAttributes, normalizes, Faraday, or HTTP client.
---

# Ruby Development

## Overview

Comprehensive Ruby and Ruby on Rails development guidelines covering code style with RuboCop and Ruby style guide, testing with RSpec, Rails environment setup, and documentation practices.

## Environment Setup

Access Rails development environment using your project's setup (Docker, local, etc.).

Common tasks to run in the development environment:
- **Rails console**: Access interactive Rails console for debugging and testing
- **Database migrations**: Run pending migrations to update database schema
- **Bundle install**: Install or update Ruby gem dependencies

When running these commands, use the appropriate method for your project (e.g., through Docker, directly on local machine, or via project-specific scripts).

### Handling Deprecation Warnings

When you encounter deprecation warnings during development or testing, follow these steps to identify and fix them.

**Identifying warnings:**

Deprecation warnings typically appear in test output or logs:
```
warning: deprecated feature X is used. Use Y instead.
```

**Reproducing warnings for debugging:**

If you need to see all deprecation warnings to fix them systematically:

```bash
# Run tests with deprecation warnings enabled
RUBYOPT="-W:deprecated" bundle exec rspec

# Or for a specific file
ruby -W:deprecated spec/models/user_spec.rb
```

**Fixing approach:**
1. Locate the source file and line number from the warning message
2. Replace deprecated syntax/API with the recommended alternative
3. Run tests again to verify the warning is resolved
4. Repeat for remaining warnings

**Best practices:**
- Fix deprecation warnings as they appear during development
- Don't modify project configuration files unless specifically required
- Use temporary `-W:deprecated` flag only when needed for debugging

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

### Ruby Style Guide

Follow Ruby community style conventions documented in the comprehensive [Ruby Style Guide](references/rubocop-ruby-style-guide.adoc).

This guide covers:
- Source code layout and formatting
- Syntax conventions and best practices
- Naming conventions
- Comments and documentation
- Classes, modules, and methods
- Collections and strings
- Regular expressions
- Metaprogramming

The style guide is maintained by the RuboCop community and reflects real-world Ruby programming practices.

## Development Best Practices

### Use Existing Helper Methods

Before implementing new logic, check for existing helper methods in the codebase:

**Where to Look:**
- Base controllers (`ApplicationController`, `AppController`, etc.)
- Concern modules
- Application helpers
- Model concerns

**Common Patterns:**
- Error rendering helpers (check for standardized error response methods)
- Type conversion helpers (boolean, integer, etc.)
- Authentication/authorization helpers
- Data formatting helpers

**Best Practice:**
```ruby
# ❌ Avoid duplicating logic
def create
  if model.save
    # ...
  else
    render json: { error: model.errors.full_messages.join(", ") }, status: :bad_request
  end
end

# ✅ Use existing helpers if available
def create
  if model.save
    # ...
  else
    render_model_error(model)  # Example: if this helper exists in base controller
  end
end
```

**Benefits:**
- Maintains code consistency
- Reduces duplication
- Centralizes common logic
- Easier to refactor and maintain

### Race Conditions in Validations

Model-level validations that check record counts or uniqueness can suffer from race conditions when multiple requests execute simultaneously.

**Problem Example:**
```ruby
# In model validation
def validate_max_records
  if associated_records.count >= MAX_COUNT
    errors.add(:base, :limit_exceeded)
  end
end

# Two simultaneous requests can both read count=29, both pass validation,
# resulting in 31 records despite MAX_COUNT=30
```

**Solution: Pessimistic Locking**

Use database-level locks to prevent race conditions:

```ruby
# In controller
def create
  ParentModel.transaction do
    parent = ParentModel.find(id).lock!  # Acquires database lock
    record = parent.associated_records.new(params)

    if record.save
      # Success
    else
      # Handle validation errors
    end
  end
end
```

**When to Use:**
- Creating records with count limits
- Checking uniqueness constraints
- Updating shared resources
- Operations requiring atomicity

**Alternative Approaches:**
- Database constraints (preferred for simple cases)
- Unique indexes for uniqueness validation
- Database check constraints for count limits
- Optimistic locking for less critical updates

**Key Principle:**
- Model validations alone cannot prevent race conditions
- Always consider concurrent access when implementing limits or constraints
- Use database-level mechanisms (locks, constraints) for critical validations

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

**Additional tips:** [rspec-additional-tips.md](references/rspec-additional-tips.md)
- Avoid duplicate `before` blocks in nested contexts

## Project Maintenance

### Documentation

After implementing features or changes:

1. Update relevant documentation in the project's documentation directory
2. Keep documentation in sync with code changes
3. Include examples and usage guidelines
4. Update API documentation if endpoints change

### Architecture Updates

When making architectural changes or adding new development conventions:

1. Document new patterns or conventions
2. Update skill files if necessary
3. Communicate changes to team

## Advanced Rails Features

### CurrentAttributes

Rails provides `CurrentAttributes` for managing per-request global state safely. See [CurrentAttributes Guide](references/current-attributes.md) for:
- Setting up request-scoped attributes
- Use cases (current user, timezone, locale)
- Thread-safety considerations
- Testing strategies

### Input Normalization

Rails 7.1+ introduces `normalizes` for automatic input sanitization. See [Input Normalization Guide](references/input-normalization.md) for:
- Using `normalizes` for automatic trimming, downcasing
- Handling nil and empty values
- Query auto-normalization
- Backward compatibility strategies

### Frozen String Literals

Enable frozen string literals to optimize memory usage. See [Frozen String Literal Guide](references/frozen-string-literal.md) for:
- Configuration options
- Performance benefits
- Migration strategies

**⚠️ Important: Check Project Configuration First**

Before adding `# frozen_string_literal: true` to files, check if the project has **global frozen string literal enabled**:

**Indicators the project already has it enabled:**
1. Check `Dockerfile` for: `RUBYOPT="--enable-frozen-string-literal"`
2. Check `docker-compose.yml` or `.env` for: `RUBYOPT` environment variable
3. Check `.rubocop.yml` for: `Style/FrozenStringLiteralComment: Enabled: false`
4. Look for documentation like `docs/FROZEN_STRING_LITERAL.md`

**If globally enabled:**
- ❌ **DO NOT** add `# frozen_string_literal: true` magic comments
- ✅ **DO** remove auto-generated frozen string literal comments from Rails generators
- ✅ Magic comments are redundant and add unnecessary visual noise

**If NOT globally enabled:**
- ✅ Add `# frozen_string_literal: true` to new Ruby files
- ✅ Follow standard Ruby/Rails conventions

**Rationale:**
When `RUBYOPT="--enable-frozen-string-literal"` is set globally (via Docker/environment), all Ruby files automatically have frozen strings enabled. Adding per-file magic comments is redundant and violates DRY principle.

### HTTP Client Integration

Use Faraday for robust HTTP API integration. See [HTTP Client Guide](references/http-client.md) for:
- Basic and advanced Faraday usage
- Service class patterns
- Error handling and retries
- Testing with WebMock

## Resources

- [Ruby Style Guide](references/rubocop-ruby-style-guide.adoc) - Comprehensive Ruby style guide maintained by RuboCop community
- [RSpec Best Practices](references/rspec-best-practices.md) - Comprehensive RSpec testing guidelines
- [CurrentAttributes Guide](references/current-attributes.md) - Rails per-request global state management
- [Input Normalization Guide](references/input-normalization.md) - Rails 7.1+ input sanitization
- [Frozen String Literal Guide](references/frozen-string-literal.md) - Ruby memory optimization
- [HTTP Client Guide](references/http-client.md) - Faraday HTTP client usage
