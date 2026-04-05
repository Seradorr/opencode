---
name: python
description: Apply Python rules for typing, packaging, testing, automation, and maintainable runtime behavior.
---

# Python

## When to use me

Use this skill for:

- Python applications and libraries
- CLI tools and automation
- test updates with pytest
- packaging and project-structure decisions

## What I enforce

- clear names and small functions
- type hints on public APIs
- `pathlib` over string-built paths where practical
- specific exceptions instead of bare `except`
- fixtures and parametrization for tests when they add signal

## Critical checks

- avoid mutable default arguments
- avoid hidden global state
- keep IO, parsing, and business logic separated
- preserve virtual-environment and packaging conventions already used by the repo
