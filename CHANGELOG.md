# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.6.0]

Loadability pass on the skill's own structure. **No prompting guidance changed in this release** — the Claude Opus 5 baselines shipped in 0.5.0 are untouched, and no component, tier, dialogue cap, or activation rule was modified. What changed is whether an agent actually reads the guidance, on the principle that content which is present but never loaded is indistinguishable from content that does not exist.

`framework.md` had grown to roughly 7400 tokens — nearly double the 4000-token reference-file ceiling declared in `CONTRIBUTING.md` — putting it at risk of being truncated or silently skipped by the loading agent. Three further failure modes surfaced alongside it: nine reference paths in `SKILL.md` were written without the `references/` prefix, so they resolve against the agent's working directory rather than the skill folder and open nothing; no file stated which reference to load when; and nothing would have caught any of it, because the budget lived in a prose guideline that also explicitly excused the file that broke it.

### Added

- `skills/prompt-best-practices/references/component-definitions.md`: the formal definitions of components 1-7 (source quote, format, what to check for, present/missing criteria, frontier-model notes), split out of `framework.md`. Pairs with `component-rubrics.md`: definitions say what a component *is*, rubrics say how to *count* it.
- `skills/prompt-best-practices/references/maintenance.md`: the verification record (source fingerprint, models verified, date), the additional source mapping, and the update procedure — split out of `framework.md` and marked contributor-only, so ~900 words of release plumbing no longer load on the runtime path. Its new step 11 makes the file-budget check part of every update.
- `skills/prompt-best-practices/SKILL.md` > **Reference Map**: a load-when table covering every reference file, a note that `maintenance.md` is contributor-only, an instruction to read a file rather than answer from memory of it, and an explicit fallback — if a reference cannot be read, complete the workflow with `SKILL.md` alone and say so in one line, rather than declining or skipping the diagnosis.
- `skills/prompt-best-practices/references/framework.md` > **"The 7 components at a glance"**: a one-line-per-component table, so Step 0 and Step 1 can run without loading a second file. The same file now carries a sibling copy of the Reference Map for agents that enter through it rather than through `SKILL.md`.
- `tests/activation-fixtures.md` > **fixture set F (loadability)**: F1 token budgets, F2 the `## When to Use` header on every reference file, F3 that every `references/…` path named in `SKILL.md` resolves — three copy-pasteable shell commands, all three verified green on this release — plus F4, the behavioral check that a missing reference degrades gracefully instead of aborting the workflow.
- `CONTRIBUTING.md` > **Budget check**: the `wc -w` command with the per-file limits, cross-referenced to fixture set F.
- `skills/prompt-best-practices/references/component-definitions.md`: frontier-model note on Component 6 — on Opus 5 scope and delegation limits earn their place, while verification clauses must be removed rather than softened.

### Changed

- `skills/prompt-best-practices/references/framework.md`: **split into three files** to bring it back inside the reference budget. It keeps the task tiers, the source-mapping table, the frontier-model calibration, the final prompt template and the adaptation guidelines; the component definitions moved to `component-definitions.md` and the maintenance material to `maintenance.md`. Measured:

  | File | Before | After |
  |---|---|---|
  | `framework.md` | ~7400 tokens | ~3270 tokens |
  | `component-definitions.md` | — | ~3530 tokens |
  | `maintenance.md` | — | ~1920 tokens |

  Every reference file now fits the 4000-token ceiling and can be loaded in full; `SKILL.md` sits at ~2900 tokens against its 5000-token limit.
- `skills/prompt-best-practices/SKILL.md`: every reference path now carries the `references/` prefix, making nine previously ambiguous pointers resolvable.
- Cross-references repointed after the split: `component-rubrics.md` (5 occurrences), `claude-considerations.md`, `codex-considerations.md`, `examples-code.md`, `examples-content.md`. References to `framework.md` > "Calibrating for frontier models", Task Tiers, Source mapping, and Final Prompt Template still resolve and were left alone; every `X.md` mentioned anywhere in the repo was verified to exist and every `file.md > "Section"` reference to point at a section that is actually there.
- `CONTRIBUTING.md`: the reference-file budget is now a hard ceiling with no canonical-file exemption — the previous wording excused `framework.md` for "sitting near it by design", which is how it drifted to ~7400 tokens. Added the `references/`-prefix rule, the Reference-Map completeness rule (both copies kept in sync), the section-title cross-reference rule, and `component-definitions.md` to the fixture-run trigger list.
- `AGENTS.md`: `component-definitions.md` and `maintenance.md` added to the reference list, `framework.md` marked as the entry point, `maintenance.md` marked contributor-only, the per-file budget stated, and `SKILL.md` > Reference Map named as the authoritative load-when table.
- `tests/activation-fixtures.md`: the Component 7 references in D1 and D3 repointed to `component-definitions.md`; the fixture-trigger line in the Purpose section extended to `component-definitions.md` and `component-rubrics.md`; L1 now spans A1–F4.
- `tests/benchmark-protocol.md`: a non-trivial change to `component-definitions.md` now also warrants a fresh benchmark run, alongside `framework.md`.
- `NOTICE.md`: attribution extended to the two files created by the split.
- `README.md`: the repository-structure tree names what `references/` actually contains.
- `.claude-plugin/plugin.json`: version `0.5.0` → `0.6.0`.

### Unchanged

- All prompting guidance. The 7 components, their definitions and quotes, the tier thresholds, the dialogue caps, the activation rules, the Fast-Track Exit, the rubric marks, the model-gated self-check, the divergence table, and every worked example are byte-for-byte the 0.5.0 content — the split moved text between files without rewriting it.
- Model baselines and verification date: Claude Opus 5 as the default target, Fable 5 / Mythos 5 as the highest-capability tier, Opus 4.8 as the refusal fallback, GPT-5.6 Sol on the OpenAI side. Still as verified on 2026-08-03; no source page was re-fetched for this release.

## [0.5.0]

Standards refresh for the release of **Claude Opus 5**, plus a restructure of the Claude reference files. Verified 2026-08-03 against the official guides: the new per-model [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) page, the re-fetched [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) page, and the main [prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) page. Claude Opus 5 is now the **default target** (Claude Fable 5 / Mythos 5 remains the highest-capability tier; Claude Opus 4.8 the refusal-fallback target). The 7-component framework, its source-mapping anchors, and all key quotes were re-verified and remain valid — no component was added, removed, or renamed.

The headline finding is that per-model guidance is **not additive**: Opus 5 inverts two instructions that prior guidance recommended. Verification / self-check clauses must be *removed* (they cause over-verification), and subagent delegation must be *capped* rather than encouraged. The framework now carries an explicit divergence table so the skill cannot flatten the two model columns into one Claude-wide default.

### Added

- `skills/prompt-best-practices/references/claude-opus-5.md`: new per-model file for Claude Opus 5 — eight tuning points (prompt for conciseness because `effort` does not shorten visible output; **delete** verification instructions rather than rewriting them; constrain task scope; cap subagent delegation; limit correction narration; describe narration cadence; calibrate written-deliverable length; keep thinking on), the effort posture (`high` default, `low`/`medium` as the primary cost lever, `xhigh` for demanding coding/agentic work), task-type notes (long-horizon spec-up-front, code review, vision-with-tools, office documents, long context), refusal handling, and a table of how the 7 components shift.
- `skills/prompt-best-practices/references/claude-fable-5.md`: the Fable 5 / Mythos 5 guidance extracted into its own file and re-verified against the current page, with the newly documented items added (checkpoint instruction, early-stopping and context-budget reminders, send-to-user tool, explicit long-run self-verification).
- `skills/prompt-best-practices/references/claude-considerations.md` > **"Pick the model before you tune"**: divergence table covering self-check, subagent delegation, response length, correction narration, thinking config, effort posture, and refusal fallback. Names the two most expensive migration mistakes explicitly.
- `tests/activation-fixtures.md` > **fixture set E (model-gated content)**: E1 (Opus 5 → verification clause removed, not reworded), E2 (Fable 5 long run → verifier instruction kept), E3 (Opus 5 → explicit length line even at quick tier), E4 (Opus 5 → delegation cap, not a delegation nudge). E1 and E2 must not resolve the same way — that is the regression this set protects.
- `skills/prompt-best-practices/references/framework.md` > "Calibrating for frontier models": two new points — **some legacy instructions are a delete, not a rewrite**, and **leaner is shared but the specifics are not** (guidance inverts between generations).
- `skills/prompt-best-practices/references/examples-code.md`: "What this prompt deliberately omits (Claude Opus 5 target)" note on Example 3, contrasting the investigate-before-answering constraint (kept) with a verification clause (dropped on Opus 5, kept on Fable 5 long runs).

### Changed

- `skills/prompt-best-practices/references/claude-considerations.md`: converted from a single flat model file into a **router** — a routing table (Opus 5 / Fable 5 / Sonnet 5 / Opus 4.8 and earlier), the refreshed "How the official guidance is organized" block now listing four per-model pages, the divergence table, six family-wide behaviors (XML tags, no prefill, effort instead of thinking budgets, motivation over bare rules, no reasoning reproduction, literal instruction following), a short Sonnet 5 section, a demoted Opus 4.8 section, and a rewritten update procedure. This keeps every Claude file well inside the 4000-token reference budget in `CONTRIBUTING.md` and mirrors Anthropic's own per-model page layout.
- `skills/prompt-best-practices/references/grounding-techniques.md`: technique 3 (self-check) is now **model-gated** — quotes the official Opus 5 exception verbatim, states that removal (not rewording) is the documented fix, and adds a per-target table. Techniques 1 and 2 remain model-agnostic.
- `skills/prompt-best-practices/references/framework.md`: maintenance table (`Last verified` → 2026-08-03; `Verified against` → Opus 5 default target, Fable 5 / Mythos 5 top tier, Sonnet 5, Opus 4.8 fallback) and recomputed source fingerprint, which now records four per-model pages, the two general-technique sections carrying an explicit Opus 5 exception (`leverage-thinking…`, `communication-style-and-verbosity`), and a quote drift on `use-examples-effectively` ("dramatically" dropped). Calibration point 5 split depth from length (`effort` is a dial for depth, not verbosity). Frontier-model notes added to Component 5 (Output specification gained weight on Opus 5) and extended on Component 4. Update-procedure step 5 and the additional-source-mapping line rewritten for the new file layout.
- `skills/prompt-best-practices/SKILL.md`: reference pointers updated to the router-plus-per-model layout; Step 2 priority 4 now skips the self-check constraint on Opus 5; Step 3 runtime hints updated (Opus 5 default target, Fable 5 / Mythos 5 top tier, Opus 4.8 previous generation) and given **two model-gated checks** before the prompt is presented; new "Target runtime is Claude Opus 5" edge case.
- `tests/activation-fixtures.md`: D1 runtime → Claude Opus 5 default target (reference updated to the family-wide behaviors section); D4 extended to Opus 5; L1 now spans A1–E4.
- `README.md`: baselines updated to Claude Opus 5 as the default target with the verification date, the non-additive-guidance warning added to the intro, and the model-specific tuning-notes list expanded to the three Claude files.
- `AGENTS.md`: reference-file list updated for the new layout, with the self-check gating noted on `grounding-techniques.md`.
- `skills/prompt-best-practices/references/codex-considerations.md`: the cross-reference to the shared lean-prompt principle now points at both Claude model files instead of the old section number.
- `NOTICE.md`: attribution extended to the Prompting Claude Opus 5 page and the new per-model files, and it now states that sample instruction snippets are condensed adaptations of the published samples.
- `.claude-plugin/plugin.json`: version `0.4.0` → `0.5.0`.

### Unchanged

- 7-component framework (Task, Role, Context, Examples, Output specification, Constraints, Structure) and the `/7` scoring denominator — re-verified against the current Anthropic guide, including the four per-model pages.
- All source-mapping anchors (`be-clear-and-direct`, `give-claude-a-role`, `long-context-prompting`, `use-examples-effectively`, `control-the-format-of-responses`, `add-context-to-improve-performance`, `structure-prompts-with-xml-tags`) and the additional-mapping anchors — still resolve.
- Task tiers, dialogue caps, activation rules, and the Fast-Track Exit.
- `component-rubrics.md`, `examples-content.md`, `benchmark-protocol.md` — model-agnostic; no changes required.
- OpenAI baselines (GPT-5.6 Sol / Terra / Luna, `reasoning.effort` ladder, `apply_patch`) — not re-verified in this release beyond the cross-reference fix; they remain as verified on 2026-07-20.

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
