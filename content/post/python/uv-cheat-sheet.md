---
title: "Uv Cheat Sheet"
date: 2025-03-30T15:46:43+08:00
description: UV cheat sheet covering Python versioning, project setup, dependency syncing, and CLI tool management.
tags: [python, uv]
---
# UV Cheat Sheet

This is a uv quick cheat sheet.

## Python Versions

```bash
# List available and installed Python versions
uv python list

# Install Python 3.14
uv python install 3.14
```

## Project Setup

```bash
# Create new project using Python 3.14
uv init my-project --python 3.14

# Add pandas as a dependency
uv add pandas

# Add pandas with version constraint
uv add 'pandas>=2.0.1'

# Add pytest as a dev dependency
uv add pytest --dev

# Add dependencies from requirements.txt with constraints
uv add -r requirements.txt -c constraints.txt

# Show available versions of pandas
uvx pip index versions pandas

# Install all dependencies
uv sync

# Install all + dev dependencies
uv sync --dev

# Remove pandas
uv remove pandas

# Run main.py in project environment
uv run main.py

# Run pytest in project environment
uv run pytest

# Show dependency tree
uv tree
```

## Python CLI Tool Management

```bash
# List available Python CLI tools managed by uv
uv tool list

# Add a CLI tool like ruff (e.g. formatter, linter)
uv tool add ruff

# Remove the installed tool
uv tool remove ruff

# Upgrade all installed tools
uv tool upgrade

# Run the tool (like ruff) inside uv environment
uvx ruff check .
```

## uv Maintenance

```bash
# Clear cached packages
uv cache clean

# Update uv to latest version
uv self update
```

## References

- [uv Official](https://docs.astral.sh/uv/)
