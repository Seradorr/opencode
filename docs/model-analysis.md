# OpenCode Self-Hosted Model Analysis

Updated: 2026-04-05

## Goal

This package targets a closed-network OpenCode installation that can be copied directly into `~/.config/opencode/` and then wired to a local or intranet OpenAI-compatible endpoint.

The goal is not to maximize benchmark variety. The goal is to maximize coding-agent stability with a small visible agent surface.

## OpenCode Facts That Drove The Layout

- Global config lives in `~/.config/opencode/opencode.json`.
- Global rules live in `~/.config/opencode/AGENTS.md`.
- Project or global skills are discovered from `skills/<name>/SKILL.md`.
- Markdown agent prompts can live globally, but extra markdown agents add visible agents unless they are configured as subagents.
- `@ai-sdk/openai-compatible` is the documented path for custom OpenAI-compatible providers.

Sources:

- OpenCode Rules: https://opencode.ai/docs/rules/
- OpenCode Agents: https://opencode.ai/docs/agents/
- OpenCode Skills: https://opencode.ai/docs/skills/
- OpenCode Providers: https://opencode.ai/docs/providers/
- OpenCode Config: https://opencode.ai/docs/config/

## Model Assessment

### Qwen3.5-397B-A17B-FP8

Why it is the default build model:

- Official model card positions it as a strong agentic and coding model.
- It exposes a native 262,144-token context window.
- It publishes strong coding-agent benchmark numbers, including SWE-bench Verified, SWE-bench Multilingual, Terminal Bench 2, MCP-Mark, and BrowseComp.
- Qwen also maintains a terminal coding agent, which is a good signal for repo-scale coding workflows.

Tradeoff:

- It is powerful but expensive and slower than the smaller routing models, so it should stay the primary implementation model rather than the background scout.

Sources:

- https://huggingface.co/Qwen/Qwen3.5-397B-A17B
- https://qwen.ai/

### Kimi-K2.5

Why it is the default plan model:

- Moonshot documents direct OpenCode usage and recommends Kimi K2.5 for programming-tool scenarios.
- Official docs position it as a strong agent and long-context reasoning model.
- It is better suited to architectural analysis and longer written outputs than to being the default edit-heavy build model.

Tradeoff:

- It is excellent for planning, but it does not need to be the default editor when Qwen 397B is already available.

Sources:

- https://platform.moonshot.ai/docs/guide/agent-support.en-US
- https://platform.moonshot.ai/docs/guide/use-kimi-k2-thinking-model.en-US

### GLM-4.7-FP8

Why it is the hidden explore model:

- Official material emphasizes improvements in multilingual agentic coding, terminal tasks, and tool use.
- The model card explicitly discusses OpenAI-style tool descriptions and agentic deployment patterns.
- It is a good fit for read-only repo reconnaissance, dependency discovery, and short fact-gathering subtasks.

Tradeoff:

- It is strong enough to help the primary agents, but the package keeps it out of the visible surface to preserve simplicity.

Sources:

- https://huggingface.co/zai-org/GLM-4.7
- https://docs.z.ai/

### Qwen3.5-35B-A3B-FP8

Why it remains the small model:

- It keeps a large context window while reducing latency and cost.
- It is useful for lightweight background tasks and future low-latency routing, even if this package keeps the visible agent surface small.

Source:

- https://huggingface.co/Qwen/Qwen3.5-397B-A17B

## Manual Alternative Models Kept In The Provider List

These models remain selectable but are not the defaults:

- `Qwen3-Next-80B-A3B-Instruct`
- `Qwen3-30B-A3B-Instruct-2507`
- `Qwen3-Coder-30B-A3B-Instruct`
- `openai/gpt-oss-120b`
- `google/gemma-3-27b-it`

Reason:

- They can still be useful for manual comparison, fallback, or cost-sensitive work.
- They should not displace the default routing without task-specific evidence.

## Community Signal And Stability Risks

### Open source tool-calling is still uneven

OpenCode issue `#234` documents failures with open-source models in agent workflows, including:

- tool-name case mismatches
- text-only responses instead of real tool calls
- inconsistent behavior across local providers

Source:

- https://github.com/anomalyco/opencode/issues/234

### Qwen3-Coder-30B is promising but not a safe default

An LM Studio issue reports `Qwen3-Coder-30B-A3B-Instruct` emitting command-style tool output instead of the OpenAI-compatible JSON most agent shells expect.

That does not make the model unusable, but it does make it risky as the package default in a closed-network coding-agent setup.

Source:

- https://github.com/lmstudio-ai/lmstudio-bug-tracker/issues/825

### Reddit signal

There is at least one recent LocalLLaMA anecdote saying Kimi K2.5 works extremely well in OpenCode. This is a weak signal, not a primary source, but it aligns with Moonshot's official positioning of Kimi for programming tools.

Source:

- https://www.reddit.com/r/LocalLLaMA/comments/1r03xow/i_tested_kimi_k25_against_opus_i_was_hopeful_and/

### Stack Overflow signal

Direct OpenCode-specific Stack Overflow coverage is thin. The useful generic lesson is that OpenAI-compatible integrations often fail because of endpoint mismatch between chat completions, responses, and completions APIs.

This package therefore stays on a single documented assumption: one OpenAI-compatible chat-completions endpoint behind `@ai-sdk/openai-compatible`.

## Routing Decision

Final routing for this package:

- `build` -> `Qwen3.5-397B-A17B-FP8`
- `plan` -> `Kimi-K2.5`
- `explore` -> `GLM-4.7-FP8`
- `small_model` -> `Qwen3.5-35B-A3B-FP8`

Why this is the package default:

- strong build quality
- strong planning quality
- only one hidden helper subagent
- broad model pool still available without cluttering the visible agent list
- reduced exposure to fragile tool-calling defaults from experimental alternatives
