---
name: ai-toolkit
description: Tune OpenCode, self-hosted model routing, prompt design, and OpenAI-compatible provider configuration.
---

# AI Toolkit

## When to use me

Use this skill when the task is about:

- OpenCode configuration
- self-hosted coding-assistant setup
- model routing and default model selection
- OpenAI-compatible endpoint wiring
- prompt, instruction, or skill design

## What I optimize for

- stable tool-calling behavior over benchmark-only choices
- clear separation between runtime instructions and reference documents
- minimal visible agent surface with strong default routing
- configs that can be copied directly into `~/.config/opencode/`

## Rules

- Keep provider IDs and model IDs stable unless the user explicitly wants a migration.
- Prefer a small number of visible agents and push specialization into skills and prompts.
- Keep coding models and non-coding models separate in the provider list.
- Default runtime docs should stay short; deep research belongs in `docs/`.
- Treat endpoint and API key handling as local config concerns, not application code concerns.
