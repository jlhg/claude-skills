# Conventional Commits Type Reference

## Available Types

### build
Changes that affect the build system or external dependencies.

**Examples:** gulp, broccoli, npm, webpack configurations

### chore
Other changes that don't modify src or test files.

**Examples:** scripts, config files, tooling updates

### ci
Changes to CI configuration files and scripts.

**Examples:** Travis, Circle, BrowserStack, SauceLabs, GitHub Actions, Husky

### docs
Documentation only changes.

**Examples:** README, API documentation, comments

### feat
A new feature.

**Examples:** user authentication, payment processing, image gallery

**Scope examples:** user, payment, gallery, auth, api

### fix
A bug fix.

**Examples:** authentication bugs, data validation issues, UI glitches

**Scope examples:** auth, data, validation, ui

### perf
A code change that improves performance.

**Examples:** query optimization, caching implementation, algorithm improvements

**Scope examples:** query, cache, algorithm, database

### refactor
A code change that neither fixes a bug nor adds a feature.

**Examples:** code restructuring, utilities reorganization, helper function updates

**Scope examples:** utils, helpers, core, services

### revert
Reverts a previous commit.

**Examples:** reverting a broken feature, undoing a problematic change

**Scope examples:** query, utils, auth

### style
Changes that do not affect the meaning of the code.

**Examples:** white-space, formatting, missing semi-colons, code beautification

**Scope examples:** formatting, linting

### test
Adding missing tests or correcting existing tests.

**Examples:** unit tests, e2e tests, integration tests

**Scope examples:** unit, e2e, integration

### i18n
Internationalization changes.

**Examples:** locale files, translation updates, language support

**Scope examples:** locale, translation, language

## Usage Examples

```
feat(auth): add social login support
fix(validation): correct email regex pattern
docs(api): update authentication endpoints
perf(query): optimize user search algorithm
refactor(utils): extract common date formatting logic
test(unit): add tests for payment processing
ci(github): add automated deployment workflow
build(deps): upgrade React to v18
chore(config): update ESLint rules
style(formatting): apply Prettier formatting
i18n(locale): add Japanese translation
revert(auth): revert "add social login support"
```

## Scope Guidelines

- Use lowercase for scopes
- Keep scopes short and meaningful
- Use consistent scopes across the project
- Optional but recommended for clarity
- Examples: auth, api, ui, db, config, core
