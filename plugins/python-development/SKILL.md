---
name: python-development
description: Python development conventions and packaging guidelines. Automatically load this skill when working with Python code or Python projects, including writing Python scripts, creating packages, publishing to PyPI with hatchling, managing dependencies, or any Python development work - even if "Python" is not explicitly mentioned in the request.
---

# Python Development

## Overview

Python development guidelines focusing on package configuration and PyPI publishing best practices.

## PyPI Publishing

### Hatchling Configuration

When using hatchling as the build backend, configure core metadata version to avoid twine validation errors.

**Required configuration in `pyproject.toml`:**

```toml
[tool.hatch.build.targets.wheel]
core-metadata-version = "2.1"

[tool.hatch.build.targets.sdist]
core-metadata-version = "2.1"
```

### License Field Format

Use simple string format for the license field:

```toml
[project]
license = "MIT"
```

**Not:**
```toml
[project]
license = {text = "MIT"}  # Avoid this format
```

### Complete Example

Minimal `pyproject.toml` for PyPI publishing:

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "your-package"
version = "0.1.0"
description = "Your package description"
authors = [
    {name = "Your Name", email = "your.email@example.com"}
]
license = "MIT"
readme = "README.md"
requires-python = ">=3.8"
dependencies = []

[project.urls]
Homepage = "https://github.com/yourusername/your-package"

[tool.hatch.build.targets.wheel]
core-metadata-version = "2.1"

[tool.hatch.build.targets.sdist]
core-metadata-version = "2.1"
```

## Publishing Workflow

### Build and Upload

```bash
# Install build tools
pip install build twine

# Build distribution packages
python -m build

# Check the distribution
twine check dist/*

# Upload to TestPyPI (optional, for testing)
twine upload --repository testpypi dist/*

# Upload to PyPI
twine upload dist/*
```

### Version Management

Update version in `pyproject.toml` before each release:

```toml
[project]
version = "0.2.0"  # Increment version
```

Or use dynamic versioning with hatchling:

```toml
[project]
dynamic = ["version"]

[tool.hatch.version]
path = "src/your_package/__init__.py"
```

## Common Issues

### Twine Check Errors

**Issue:** `twine check` fails with metadata validation errors

**Solution:** Ensure `core-metadata-version = "2.1"` is set for both wheel and sdist targets

### License Format Errors

**Issue:** PyPI rejects package due to license field format

**Solution:** Use simple string format: `license = "MIT"` instead of object format

## Best Practices

- Always run `twine check` before uploading
- Test on TestPyPI before publishing to production PyPI
- Keep dependencies minimal in `pyproject.toml`
- Use semantic versioning (MAJOR.MINOR.PATCH)
- Include a comprehensive README.md
- Add project URLs (homepage, documentation, repository)
- Specify Python version compatibility with `requires-python`
