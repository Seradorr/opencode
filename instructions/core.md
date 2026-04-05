# Core Workflow

- Read the affected files before editing them.
- Prefer the smallest change that satisfies the request.
- Do not bundle unrelated refactors into functional work.
- When several files are involved, identify the dependency order first and then edit in that order.
- Validate with the cheapest relevant command after meaningful edits.
- If a command or edit fails twice, change strategy instead of repeating the same action.
- Use the `explore` subagent only for broad, read-only discovery. Do not delegate the immediate blocking implementation step.
