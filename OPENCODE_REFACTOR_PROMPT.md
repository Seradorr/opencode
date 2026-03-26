# OpenCode Refactor Prompt

Use this prompt together with the attached `CONTINUE_REFACTOR_PLAYBOOK.md` file.

---

Read the attached `CONTINUE_REFACTOR_PLAYBOOK.md` first, completely. Treat it as the source of architectural intent.

Your task is to apply the same class of improvements to the OpenCode configuration, which is defined in a `.jsonc` file, plus any related rule/policy/docs files in that repository.

Do not do a blind one-to-one copy from Continue. Port the principles, not the exact names, unless the target repository clearly benefits from the same names.

## Objective

Refactor the OpenCode config so that it moves from a crowded, domain-model-heavy setup to a leaner, more professional architecture with:

- a small set of core behavior engines
- domain specialization moved into rules/policies/globs/manual rule inclusion
- separate tuning blocks kept intact per model family or role
- no loss of expertise, quality bar, or safety behavior
- docs aligned with the new architecture

The final result should feel professional and robust, at a Codex / Claude Code / Copilot level of engineering behavior.

## Research-first requirement

Before making any architectural change, do detailed research. Do not refactor from assumptions or memory.

You must read and compare all relevant sources you can access, including:

- OpenCode official documentation
- OpenCode official API or configuration documentation
- OpenCode GitHub repository files relevant to config, rules, policies, prompts, profiles, schemas, examples, and changelogs
- GitHub issues, discussions, PRs, or commit history that clarify intended behavior
- community reports and usage discussions such as Reddit, when they provide practical evidence about model behavior or configuration pitfalls

You must also fetch and inspect the actual local repository files before editing them.

This means:

- read the current `.jsonc` config fully
- read all related rule/policy/system-prompt/profile files
- inspect any local schema/examples/docs files that define valid structure
- inspect git history before changing tuning or architecture
- fetch external docs/pages as needed and use them as evidence

Do not guess what OpenCode supports. Verify it.

## What you must do

1. Research the OpenCode platform in detail using official docs, official API/config docs, GitHub sources, relevant issues/discussions, and practical community evidence where useful.
2. Inspect the current OpenCode `.jsonc` config and all related rule/policy/system-prompt files.
3. Inspect git history before changing tuning values. Especially check the history of:
   - `frequency_penalty`
   - `presence_penalty`
   - `top_p`
   - `temperature`
   - `maxTokens`
   - `contextLength`
   - truncation or prompt budget settings
   - reasoning-related settings
4. Identify which parts are:
   - duplicated across many model entries
   - really behavior-level concerns
   - really domain-specialization concerns
   - better expressed as rules/policies than model-specific prompts
5. Refactor the config into a small number of core engines, equivalent in spirit to:
   - hard engineer / high-quality deep reasoning engine
   - soft engineer / fast lightweight engine
   - vision or long-context engine if the platform supports it
   - optional alternative hard engine if there is a second useful model family
   - dedicated rerank and embedding models if present
6. Move domain specialization out of duplicated model prompts and into the repository's rule/policy layer.
7. Remove prompt-based specialization if the rule/policy system can carry that responsibility cleanly.
8. Keep separate tuning objects/blocks for different model families or roles. Do not merge everything into one shared block just for simplicity.
9. Preserve historically intentional tuning decisions unless there is strong evidence they should change.
10. Update docs so they match the new architecture exactly.
11. Validate the final JSONC and any referenced files.

## Non-negotiable design rules

- Do not lose technical knowledge while simplifying.
- Do not reduce the professionalism of the assistant.
- Do not collapse everything into one generic model if that weakens behavior control.
- Do not keep many duplicate domain-specific model entries if rules can express the same specialization more cleanly.
- Do not remove historically intentional penalties or sampling decisions without checking git history first.
- Do not claim a feature exists unless it is verified from the repository, schema, docs, or other strong evidence.
- Write rules, skills, prompts, policies, and instruction content in Turkish by default.
- Keep user-facing and behavior-defining guidance in Turkish; technical terms may remain in English where clearer.
- Do not expose secrets, organization-specific names, internal endpoints, or sensitive paths.
- Do not overwrite unrelated user changes.

## Architectural target

Aim for this structure in spirit:

- few physical model entries
- clear separation between behavior role and domain expertise
- rule-driven domain activation
- optional manual rule inclusion for non-file-based topics such as git, architecture, review, debugging, release flow, etc.
- separate model tuning blocks retained for future divergence

If OpenCode uses different concepts than Continue, map them carefully:

- Continue model entries -> OpenCode model definitions or profiles
- Continue rules -> OpenCode rules, policies, instructions, profiles, or equivalent
- Continue globs -> file pattern activation or equivalent targeting logic
- manual `@rule` style inclusion -> closest OpenCode equivalent, if any

## Important nuance

In the Continue refactor, prompt-based domain specialization was intentionally removed because rules and globs were sufficient.

Apply the same principle here:

- if OpenCode already has a strong rule/policy mechanism, use it
- if it does not, create the closest clean equivalent
- do not add a second redundant specialization layer unless there is a clear benefit

## Historical tuning requirement

You must explicitly check git history for model tuning decisions before finalizing the refactor.

If you find a parameter that was deliberately restored in the past, treat that as a strong signal that it should be preserved.

Your summary must clearly state:

- which historical tuning decisions were preserved
- which ones changed
- why

## External research requirement

Your final summary must also clearly state:

- which official docs were consulted
- which repository files or schemas were inspected
- which external discussions or community references materially influenced decisions
- which assumptions were avoided because they were verified from evidence

## Documentation requirement

If the config architecture changes, update the docs in the same refactor.

The docs must explain:

- the new core engine layout
- where domain expertise now lives
- how manual specialization is triggered, if applicable
- any important safety, git, or production-quality rules

Documentation, rule content, skill content, prompt content, and behavior-defining instruction text should be written in Turkish by default, unless the target repository has a strong existing reason to do otherwise.

Avoid docs drift.

## Output requirements

After completing the refactor, provide:

1. A short explanation of the new architecture.
2. A list of files changed.
3. A before/after explanation of how specialization now works.
4. A short section called `Historical tuning preserved`.
5. A short section called `Open risks or follow-ups`.

## Quality bar

The end state should be:

- leaner
- easier to maintain
- easier to tune
- more deterministic
- still highly capable
- still domain-aware
- still safe for git and production-oriented workflows

If there is any conflict between simplicity and preserving real expertise, preserve expertise and restructure carefully rather than over-simplifying.

Now perform the refactor.
