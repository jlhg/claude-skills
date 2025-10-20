---
name: python-development
description: Python development conventions and packaging guidelines. Automatically load this skill when working with Python code or Python projects (detected by .py files, pyproject.toml, requirements.txt, setup.py, poetry.lock, Pipfile, or __init__.py), including writing Python scripts, creating packages, publishing to PyPI with hatchling/setuptools/poetry, managing dependencies with pip/poetry/pipenv, working with virtual environments (venv/virtualenv/conda), testing with pytest/unittest, linting with pylint/flake8/ruff, formatting with black/autopep8, type checking with mypy, building web applications with Django/Flask/FastAPI, data science with pandas/numpy, or any Python development work - even if "Python" is not explicitly mentioned in the request. Apply when encountering Python-specific terms like pip, venv, pytest, __init__.py, requirements.txt, or Python package names.
---

# Python Development

## Overview

Python development guidelines focusing on package configuration and PyPI publishing best practices.

**Environment Detection**: This skill applies to Python projects, typically identified by:
- `.py` files (Python source code)
- `pyproject.toml` (modern Python project configuration)
- `requirements.txt` or `requirements-dev.txt` (pip dependencies)
- `setup.py` or `setup.cfg` (legacy package configuration)
- `poetry.lock` or `Pipfile.lock` (dependency lock files)
- `__init__.py` (Python package markers)
- Virtual environment directories (`venv/`, `.venv/`, `env/`)
- Python-specific directories (`src/`, `tests/`, `__pycache__/`)
- Configuration files for Python tools (`.flake8`, `pytest.ini`, `mypy.ini`, `pyproject.toml`)

**Common Python Frameworks & Tools**:
- Web: Django, Flask, FastAPI, Tornado, Pyramid
- Testing: pytest, unittest, nose, tox
- Package Management: pip, poetry, pipenv, conda
- Code Quality: black, pylint, flake8, ruff, mypy, isort
- Data Science: pandas, numpy, scipy, matplotlib, jupyter
- Async: asyncio, aiohttp, trio

**Important**: Always apply this skill when working in a Python project context, even if the user doesn't explicitly mention "Python" in their request.

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
