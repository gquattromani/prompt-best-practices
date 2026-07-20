# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0]

Standards refresh to the current flagship models and an installation-layout fix. Verified 2026-07-20 against the official guides: Anthropic's per-model [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) page and OpenAI's [GPT-5.6 model guidance](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6). The current Claude flagship reference is Claude Fable 5 / Mythos 5 (with Opus 4.8 as the recommended fallback target); the current OpenAI flagship is GPT-5.6 Sol (Terra / Luna siblings). The 7-component framework, its source-mapping anchors, and all key quotes were re-verified and remain valid — no component was added, removed, or renamed.

### Changed

- **Installation layout (spec compliance):** moved the skill from `skill/` to `skills/prompt-best-practices/` so the skill folder name matches the `name` frontmatter (required by the [Agent Skills specification](https://agentskills.io/specification): "Must match the parent directory name") and so it lands in the `skills/` directory that Claude Code plugins auto-discover. The previous layout (`name: prompt-best-practices` inside a folder literally named `skill`) fails `skills-ref validate`.
- `.claude-plugin/plugin.json`: version `0.3.0` → `0.4.0`; removed the now-obsolete `"skills": ["./skill"]` path — the skill is auto-discovered from `skills/`.
- `.claude-plugin/marketplace.json`: replaced the dead `$schema` URL (the old `anthropics/claude-code/.../marketplace.schema.json` path now 404s) with the resolving `https://json.schemastore.org/claude-code-marketplace.json`. Re-verified the file validates: required `name`, `owner`, `plugins[].name`, `plugins[].source` all present.
- `skills/prompt-best-practices/references/framework.md`: new **"Calibrating for frontier models"** section capturing the directive both providers now share — leaner prompts outperform padded ones. Fable 5: "Skills developed for prior models are often too prescriptive … and can degrade output quality." GPT-5.6: leaner system prompts scored ~10-15% higher while using 41-66% fewer tokens. Added a frontier-model note to Component 4 (Examples are the first component to cut). Updated the maintenance table (`Last verified` → 2026-07-20; `Verified against` → Fable 5 / Mythos 5 flagship, Opus 4.8 fallback, Sonnet 5) and recomputed the source fingerprint.
- `skills/prompt-best-practices/references/claude-considerations.md`: rewritten around Claude Fable 5 as the flagship (16 numbered points: refactor-don't-over-prescribe, brief-instruction steering, effort as the primary dial, adaptive-thinking-only + `refusal` stop reason, longer turns, grounding progress claims, stating boundaries, parallel subagents, over-engineering risk, no reasoning reproduction / `reasoning_extraction` refusal, give-the-reason, memory systems, readability addendum, safety classifiers, XML tags, prefill deprecation). Added a "Claude Opus 4.8 (fallback target)" section and a per-model-page "How the official guidance is organized" block. Rewrote the update procedure.
- `skills/prompt-best-practices/references/codex-considerations.md`: rewritten around the GPT-5.6 family. Recommended-models table → `gpt-5.6-sol` (frontier default, `gpt-5.6` alias), `gpt-5.6-terra` (balanced), `gpt-5.6-luna` (high-volume); `gpt-5.5` as previous-gen; `gpt-5.3-codex-spark` preview; `gpt-5.2` / `gpt-5.3-codex` deprecated. New headline section "outcome-first, leaner prompts" (what to remove / what to keep). Added `reasoning.effort` ladder (default `medium`), `text.verbosity`, autonomy-and-approval boundaries, multi-agent [beta] + programmatic tool calling, persisted reasoning / Pro mode / prompt caching. Kept `apply_patch` and the Markdown-or-XML structure guidance. Reframed the framework-mapping table through the lean-prompt lens.
- `skills/prompt-best-practices/SKILL.md`: Step 3 runtime hints updated to the current flagships and given a "Frontier-model calibration" note (build the smallest sufficient prompt); the OpenAI Codex edge case rewritten for the GPT-5.6 family (outcome-first, `apply_patch`, approval boundaries, `reasoning.effort`).
- `skills/prompt-best-practices/references/examples-code.md`: Example 4 Codex target `gpt-5.5` → `gpt-5.6-sol`; replaced the "no preamble/upfront plan" constraint with a single autonomy-and-approval boundary, and reframed the "what changed" notes around GPT-5.6's leaner-prompt guidance.
- `tests/activation-fixtures.md`: Fixture D1 Claude runtime → Fable 5 / Mythos 5 flagship (Opus 4.8 fallback); D2 Codex runtime → `gpt-5.6-sol`; added D4 (skill trims an over-specified prompt on frontier models).
- `README.md`: model baselines updated to Fable 5 / Mythos 5 and GPT-5.6 Sol; OpenAI reference now points to the GPT-5.6 model guidance; added the leaner-prompt framing; all `skill/…` paths updated to `skills/prompt-best-practices/…`.
- `AGENTS.md`: skill location and reference-file descriptions updated for the new layout and the current flagships.

### Unchanged

- 7-component framework (Task, Role, Context, Examples, Output specification, Constraints, Structure) and the `/7` scoring denominator — re-verified against the current Anthropic and OpenAI guides.
- All source-mapping anchors on the Anthropic prompting page (`be-clear-and-direct`, `give-claude-a-role`, `long-context-prompting`, `use-examples-effectively`, `control-the-format-of-responses`, `add-context-to-improve-performance`, `structure-prompts-with-xml-tags`) — still resolve on the three-part restructured page.
- `component-rubrics.md`, `grounding-techniques.md`, `examples-content.md`, `benchmark-protocol.md` — model-agnostic; no changes required.

## [0.3.0]

Guideline refresh after fetching the current Anthropic and OpenAI official guides (verified 2026-06-16). Anthropic restructured its documentation into per-model pages, the current Claude flagship is Opus 4.8 (with Fable 5 / Mythos 5 as newest siblings), and the current Codex flagship is `gpt-5.5`. The 7-component framework, its source-mapping anchors, and all key quotes were re-verified and remain valid — no component was added, removed, or renamed.

### Changed

- `skill/references/claude-considerations.md`: replaced `## Claude 4.7` with `## Claude Opus 4.8` and rewrote the numbered guidance against the new dedicated [Prompting Claude Opus 4.8](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8) page (literal instruction following at low effort, response-length calibration, direct/opinionated tone, the full `low`→`max` effort ladder with `xhigh` as the coding/agentic default, adaptive thinking off by default, reasoning-over-tools bias, fewer subagents and better progress updates, over-engineering risk, code-review harness re-tuning, the persistent cream/serif/terracotta frontend house style, interactive-vs-autonomous token use, prefill deprecation). Added a "How the official guidance is organized" section explaining that model-specific guidance now lives on per-model pages, plus a pointer to the [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) page (Fable 5 / Mythos 5). Rewrote the update procedure for the per-model-page structure.
- `skill/references/codex-considerations.md`: updated the "Recommended models" table to `gpt-5.5` (new default — "start with `gpt-5.5`"), `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex-spark`, with `gpt-5.2` and `gpt-5.3-codex` noted as deprecated for ChatGPT sign-in. Point 1 (preambles) rewritten: upfront-plan/preamble suppression is now model-version dependent — `gpt-5.3-codex`+ encourage cadence-based progress updates (every 1-3 steps; floor every 6 steps / 10 tool calls), so "no upfront plans" is no longer a blanket rule. Refreshed the framework-mapping table (Role → *Autonomy and Persistence*, Output spec → *Presenting your work and final message*) and noted the new *Plan tool* / *Special user requests* / *Frontend tasks* Starter Prompt sections. Updated the "verified" date to 2026-06-16.
- `skill/references/framework.md` > maintenance table: `Last verified` → 2026-06-16; `Verified against` → Claude Opus 4.8 (default) and Claude Sonnet 4.6, with Fable 5 / Mythos 5 noted as newest siblings; recomputed the source fingerprint to record the three-part page restructure, the move of model-specific guidance to dedicated pages, the stable general-principles anchors, and the new "Communication style and verbosity" subsection.
- `skill/references/framework.md` > Update procedure step 5: now reflects that model-specific guidance lives on per-model `prompting-claude-<model>` pages rather than in-page anchors.
- `README.md`: tuning-notes baselines updated to Opus 4.8 (Fable 5 / Mythos 5 aware) and `gpt-5.5`; the Codex line now says preamble cadence is model-version dependent instead of "no upfront plans".
- `skill/references/examples-code.md`: Example 4 Codex target bumped from `gpt-5.4` to `gpt-5.5`.
- `tests/activation-fixtures.md`: Fixture D1 Claude runtime → Opus 4.8 (and `claude-considerations.md` §6 → §8 for the XML-tags point), Fixture D2 Codex runtime → `gpt-5.5`.

### Unchanged

- 7-component framework (Task, Role, Context, Examples, Output specification, Constraints, Structure) and the `/7` scoring denominator — re-verified against the current Anthropic guide for Opus 4.8.
- All source-mapping anchors (`be-clear-and-direct`, `give-claude-a-role`, `long-context-prompting`, `use-examples-effectively`, `control-the-format-of-responses`, `add-context-to-improve-performance`, `structure-prompts-with-xml-tags`) and the additional-mapping anchors (`overthinking-and-excessive-thoroughness`, `overeagerness`, `tool-usage`, `minimizing-hallucinations-in-agentic-coding`, `leverage-thinking--interleaved-thinking-capabilities`) — all still resolve on the restructured page.
- `SKILL.md`, `component-rubrics.md`, `grounding-techniques.md`, `examples-content.md` — model-agnostic; no changes required.

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
