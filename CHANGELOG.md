# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0]

### Added

- `skill/references/codex-considerations.md`: new reference file with OpenAI Codex-specific guidance, including current model list (`gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex`, `gpt-5.3-codex-spark`), framework mapping to the official Codex Starter Prompt sections, and behaviors that differ from Claude (no upfront plans, explicit parallelization, `apply_patch` format, phase-aware output, dirty worktree handling, autonomy bias). Point 7 added later during the audit states Markdown headings and XML are both valid Structure formats on Codex.
- `skill/references/grounding-techniques.md`: extracted the three grounding-and-accuracy techniques (investigate before answering, ground in quotes, self-check) into a dedicated reference file so `framework.md` stays within the 3000-token budget defined in `CONTRIBUTING.md`.
- `skill/references/examples-code.md`: new dialogue example showing the final prompt adapted for OpenAI Codex (apply_patch format, explicit parallelization, dirty-worktree Constraint, no upfront plan), illustrating the multi-platform claim in the README.
- `skill/references/component-rubrics.md`: operational checklist that turns `[OK] / [~~] / [--]` component classification into a deterministic rubric. Each of the 7 components has named required checks (e.g., Task's T1 "imperative verb" and T2 "success criterion") so two runs on the same prompt produce the same diagnostic. Addresses the non-deterministic classification issue surfaced by the audit.
- `tests/activation-fixtures.md`: regression suite with reference prompts and expected activation outcomes (`ACTIVATE_FULL`, `ACTIVATE_LIGHT`, `FAST_TRACK`, `DO_NOT_ACTIVATE`) across auto-mode, slash-command, and edge-case fixture sets. Contributors run these before merging changes to `SKILL.md`, `framework.md`, or `component-rubrics.md`.
- `tests/benchmark-protocol.md`: protocol for measuring the framework's effect on output quality. Defines the prompt set, the baseline vs. optimized procedure, the judging rubric, the "significantly better" threshold, and an honesty clause requiring negative results to be published.
- `skill/references/framework.md` > Task Tiers: explicit quick / standard / complex tier definitions with required-components count and dialogue-cap mapping. The tiers operationalize the "modular, not prescriptive" principle into a concrete rule `SKILL.md` can apply.
- `skill/references/framework.md` > Component 7 > format options table: Structure is now explicitly format-agnostic with XML, Markdown headings, JSON, and fenced blocks listed as valid. Claude defaults to XML; Codex works with either.
- `skill/references/framework.md` > Component 5 > Tone/Avoid coexistence note: explains why positive framing (`Tone`) and anti-patterns (`Avoid`) are not a contradiction — the former is primary direction, the latter a targeted disambiguator.
- `skill/references/framework.md` > maintenance table: added a `Source fingerprint` row summarizing the sections and quotes the framework depends on, so drift from the Anthropic guide is visible at a glance.
- `skill/SKILL.md` Non-goals section: states the skill is not a generic writing coach, not an empirical guarantee, and not XML-only.
- `skill/SKILL.md` Edge Cases: added a "Target model is OpenAI Codex" entry pointing to `codex-considerations.md` and an "invoked by mistake" entry pointing to the Fast-Track Exit.
- `README.md` > Non-goals section and `CONTRIBUTING.md` > Scope and non-goals section: explicit list of what the project is not, intended to slow scope creep.
- `AGENTS.md` entries for `component-rubrics.md`, `activation-fixtures.md`, and `benchmark-protocol.md`.
- `tests/activation-fixtures.md` > Fixture set D: runtime-specific Structure format selection (Claude → XML, Codex → XML or Markdown, unknown → XML).
- "When to Use" opening section on every reference file (`framework.md`, `grounding-techniques.md`, `examples-content.md`, `examples-code.md`, `claude-considerations.md`, `codex-considerations.md`, `component-rubrics.md`), aligning the files with the pattern stated in `CONTRIBUTING.md`.
- `SECURITY.md`: private-reporting policy for vulnerabilities and prompt-injection patterns; Security Review section appended with the results of the 2026-04-20 manual audit (seven categories of risky patterns searched and not found).
- `CODE_OF_CONDUCT.md`: Contributor Covenant v2.1 for public-repository community standard.
- `NOTICE.md`: third-party attributions (Anthropic prompting best practices, OpenAI Codex Prompting Guide, Contributor Covenant v2.1 under CC BY 4.0) and a trademark disclaimer listing Claude, Anthropic, OpenAI, Codex, ChatGPT, GitHub Copilot, Cursor, Windsurf, Roo Code, Goose, and Gemini CLI as marks of their respective owners with an explicit statement that the project is not affiliated with or endorsed by any of them.
- `LICENSE`: added the missing copyright holder name (`Giandemetrio Quattromani`) so the MIT license notice is complete.
- `README.md`: new "Contributing" section linking to `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, and `SECURITY.md`; updated repository structure to list the new files, the `tests/` directory, and `NOTICE.md`; License section now points to `NOTICE.md` and includes an inline trademark disclaimer.
- `CONTRIBUTING.md`: replaced the inline one-line Code of Conduct with a pointer to `CODE_OF_CONDUCT.md`, and added a Security section pointing to `SECURITY.md`.

### Changed

- `skill/SKILL.md`: rewritten activation rules around task tiers rather than a flat 3-component threshold. Auto-activation now requires (a) an execution request, (b) standard-or-complex tier, and (c) components below tier threshold — micro-tasks and questions are explicitly excluded. Slash-command invocation still mandatorily assesses, but a documented Fast-Track Exit (`skip`, `go`, `execute`, `as-is`) lets users exit at Step 1 without the dialogue. Step 2 is capped by tier: quick=2 questions, standard=3, complex=5.
- `skill/SKILL.md`: consolidated duplicate activation rules and resolved a logical conflict in the Edge Cases. Removed the standalone "fewer than 3 components" line, four redundant Edge Cases ("User wants to skip", "already good", "excellent", "Code-related requests"), and trimmed the Step 0 fallback line so it points to the Activation Rules instead of restating them.
- `skill/SKILL.md`: updated the Step 2 Priority 4 note to point to `grounding-techniques.md` instead of the now-deleted Grounding section inside `framework.md`.
- `skill/references/claude-considerations.md`: replaced `## Claude 4.6` with `## Claude 4.7` and refreshed guidance for the new model (literal instruction following, response length auto-calibration, tone shift, `xhigh` effort and adaptive thinking, reduced default tool use and subagent spawning, code review harness tuning, frontend design defaults, prefilled response deprecation). Point 6 qualifies XML as a Claude-specific recommendation rather than a universal one.
- `skill/references/framework.md`: updated maintenance metadata to `Last verified: 2026-04-20` and `Verified against: Claude 4.7 (Opus), Claude 4.6 (Sonnet)`. Extended the adaptation guidelines to reference `claude-considerations.md`, `codex-considerations.md`, `grounding-techniques.md`, and `component-rubrics.md`. Removed the inline Grounding section (extracted to its own file).
- `skill/references/framework.md` > Component 7 (Structure): rewritten to be format-agnostic with a runtime-to-format matrix. XML remains the Claude default, Codex accepts both XML and Markdown; no single format is presented as the universal best choice.
- `skill/SKILL.md` > Step 3 (Build Final Prompt): runtime-to-format selection lists Claude (XML) and Codex (XML or Markdown).
- `README.md`: intro paragraph explicitly names "Anthropic Claude and OpenAI Codex" as first-class runtimes.
- `skill/references/framework.md` > Update procedure: added fingerprint recomputation and mandatory re-run of activation fixtures. Step count grew from 8 to 11 to cover `component-rubrics.md` and the regression suite.
- `skill/references/framework.md` > Adaptation Guidelines: now points to the canonical Task Tiers table instead of restating the mapping inline.
- `skill/references/examples-content.md` and `skill/references/examples-code.md`: split the former single `examples.md` into two topic-scoped files (content/communication vs. code/engineering) to stay within the 3000-token reference budget and match the "one topic per file" guideline in `CONTRIBUTING.md`.
- `README.md`: softened the first-pass-correctness claim ("reduces" instead of "eliminates"), added Non-goals, pointed to `component-rubrics.md` and `tests/` in the How-It-Works and Repository-Structure sections, and clarified that the framework is most thoroughly tested on Claude. Repository structure note now mentions model-specific guidance files and the `tests/` directory. Rewrote the Uninstall section to resolve a contradiction between the stated multi-platform compatibility and the previously Claude-Code-only uninstall instructions — now explicitly branched by install method (Claude Code plugin vs. `skills.sh` / other agents).
- `CONTRIBUTING.md`: added Scope-and-non-goals and updated the PR process to require running `activation-fixtures.md` before merge; large behavior changes should also include a `benchmark-protocol.md` result.

### Removed

- `.claude-plugin/marketplace.json`: removed the vestigial top-level `version` field. The authoritative version for the plugin lives in `.claude-plugin/plugin.json`; the marketplace-level field is not used by Claude Code and could interfere with auto-updates.
- `skill/references/examples.md`: replaced by `examples-content.md` and `examples-code.md`.

### Unchanged

- 7-component framework (Task, Role, Context, Examples, Output specification, Constraints, Structure) — confirmed valid for Opus 4.7 by the official Anthropic guide.
- Scoring denominator (`/7`) across all examples.
- The two dialogue-example files (`examples-content.md`, `examples-code.md`) — their scoring denominator and component-by-component breakdown remain valid. The final prompts still apply the template from `framework.md`.

## [0.1.0]

### Added

- Interactive prompt structuring skill using a 7-component framework derived from Anthropic's official prompting best practices. Each component maps to a specific section of the documentation.
- 4-step workflow: Quick Assessment, Diagnosis, Guided Dialogue (one question at a time), Build Final Prompt. Optional Step 4 for refinement.
- Fast path for experienced users: skip dialogue for components already provided.
- Language detection: responds in the same language as the user's prompt.
- Model-agnostic language throughout instructional text. Claude-specific guidance isolated in `references/claude-considerations.md`.
- 6 dialogue examples (3 complete with final prompt, 2 abbreviated, 1 no-activation) and 3 anti-pattern examples.
- Grounding and accuracy section with 3 techniques for hallucination prevention.
- Plugin manifest (`.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`) and Agent Skills standard support (`AGENTS.md`).
- Reference files: `framework.md` (component definitions, source mapping, prompt template, update procedure), `examples.md` (dialogue examples), `claude-considerations.md` (Claude 4.6 specific tips).
