# Activation Regression Fixtures

## Purpose

Reference prompts with expected activation outcomes. A contributor modifying `SKILL.md`, `framework.md`, `component-definitions.md`, or `component-rubrics.md` must verify that every fixture still produces its expected outcome. This is the regression test for the skill's behavior.

These fixtures address the gap flagged in `SECURITY.md` > Scope > "Broken activation rules". Run them manually against the skill (or via an automated harness if one is added later); they are the contract the skill has to honor.

## How to run

For each fixture:

1. Invoke the skill as described in the `Trigger` column (slash command or auto-mode prompt).
2. Observe whether the skill activates, fast-tracks, or declines.
3. Compare against `Expected outcome`.
4. If any fixture fails, the change is a regression — fix or document before merging.

## Legend

- `ACTIVATE_FULL` — skill runs Step 1 diagnosis and full dialogue.
- `ACTIVATE_LIGHT` — skill runs Step 1, but the tier cap limits the dialogue to ≤2 questions.
- `FAST_TRACK` — skill runs Step 1 and proceeds to execute without dialogue (e.g., because the user said "skip").
- `DO_NOT_ACTIVATE` — skill does not run; the host agent answers directly.

---

## Fixture set A — Auto-mode prompts (no slash command)

| # | Prompt | Tier | Expected outcome | Rationale |
|---|---|---|---|---|
| A1 | `Rename the variable userList to users in UserViewModel.swift` | Quick (micro-task) | `DO_NOT_ACTIVATE` | Precise code-level micro-task. Codebase supplies context. |
| A2 | `Fix the typo in line 42 of README.md` | Quick (micro-task) | `DO_NOT_ACTIVATE` | Typo fix. No ambiguity. |
| A3 | `What is the Observer pattern in Swift?` | N/A (question) | `DO_NOT_ACTIVATE` | Conversational question, not an execution request. |
| A4 | `Write me an email` | Quick → Standard (depends on stakes) | `ACTIVATE_FULL` | Execution request; no success criterion; no output shape. |
| A5 | `Write a commit message for the staged diff` | Quick | `ACTIVATE_LIGHT` | Execution request; tier cap limits dialogue to ≤2 questions. |
| A6 | `Build me an endpoint to reset user passwords` | Complex | `ACTIVATE_FULL` | Security-sensitive, multi-subsystem, needs full dialogue. |
| A7 | `Migrate the auth middleware from sessions to JWT` | Complex | `ACTIVATE_FULL` | Cross-cutting refactor; stakes high. |
| A8 | `Just write me a haiku about autumn, don't overthink` | Quick | `DO_NOT_ACTIVATE` | User explicitly opted out of optimization. |
| A9 | `Add the missing import for React in App.tsx` | Quick (micro-task) | `DO_NOT_ACTIVATE` | Trivial, no ambiguity. |
| A10 | `<role>You are a senior Swift engineer</role><task>Write tests for PaymentService with 100% branch coverage</task><output_spec>Swift Testing, clear test names</output_spec><constraints>Follow tests/OrderServiceTest.swift pattern</constraints>` | Complex (already structured) | `DO_NOT_ACTIVATE` | 5+ components present at `[OK]`; meets complex-tier threshold. |

## Fixture set B — Slash-command invocation

The `MANDATORY on slash-command invocation` rule applies. Step 0 always runs; Fast-Track Exit is offered in Step 1.

| # | Prompt | Expected outcome | Rationale |
|---|---|---|---|
| B1 | `/prompt-best-practices write me an email` | `ACTIVATE_FULL` | Standard tier, low structure. |
| B2 | `/prompt-best-practices rename foo to bar` | `ACTIVATE_LIGHT` | Slash = assess; tier = quick, cap 2 questions. Fast-Track Exit offered. |
| B3 | `/prompt-best-practices what is recursion?` | `ACTIVATE_LIGHT` | Slash = assess; skill should note the prompt is a question and offer to skip. |
| B4 | `/prompt-best-practices` (user then replies `skip` to Step 1) | `FAST_TRACK` | Fast-Track Exit honored. |
| B5 | `/prompt-best-practices <full 7-component prompt>` | `ACTIVATE_LIGHT` | Slash = assess; diagnostic shows all `[OK]`; skill should immediately offer to execute. |

## Fixture set C — Ambiguity and edge cases

| # | Prompt | Expected outcome | Rationale |
|---|---|---|---|
| C1 | `Write me an email. Just execute, no questions.` | `DO_NOT_ACTIVATE` | Explicit opt-out. |
| C2 | `Write me an email about the Q2 numbers. It should be under 200 words, formal tone, and get them to confirm the meeting on Thursday.` | Standard, meets threshold | `DO_NOT_ACTIVATE` | Task, Output spec, Success criterion all present → ≥3 components. |
| C3 | `Analyze these sales data and give me useful insights` | Complex (open-ended) | `ACTIVATE_FULL` | "Useful" is not a success criterion; many unknowns. |
| C4 | `Debug why the integration tests are flaky` | Complex | `ACTIVATE_FULL` | Open-ended investigation; benefits from scoping. |
| C5 | `Add a null-check at line 88 of UserService.ts` | Quick (micro-task) | `DO_NOT_ACTIVATE` | Precise, small, context from codebase. |
| C6 | `Refactor UserService to be cleaner` | Standard / Complex | `ACTIVATE_FULL` | "Cleaner" is subjective; needs success criterion. |
| C7 | User starts dialogue, gives one answer, then replies `let's go` to next question. | Dialogue stops early | Skill must honor impatience signal and build with what it has. |

## Fixture set D — Runtime-specific behavior

These fixtures verify that Step 3 (Build Final Prompt) picks the right Structure format per runtime.

| # | Context | Expected format in final prompt | Rationale |
|---|---|---|---|
| D1 | Target runtime: Claude (Opus 5 default target, Claude Code) | XML tags | `component-definitions.md` > Component 7 default; `claude-considerations.md` > "Behaviors shared across the current family" § 1. |
| D2 | Target runtime: Codex (`gpt-5.6-sol`) | Markdown headings or XML, both acceptable | `codex-considerations.md` > "Other Codex-specific behaviors" § 4. |
| D3 | Target runtime not specified | XML tags (safe default) | `component-definitions.md` > Component 7 > Runtime-specific defaults. |
| D4 | Target runtime: Claude Opus 5, Claude Fable 5, or GPT-5.6, prompt over-specified (padded examples, repeated rules) | Skill trims to the smallest sufficient prompt | `framework.md` > "Calibrating for frontier models"; leaner prompts win on all current frontier models. |

## Fixture set E — Model-gated content

These fixtures verify that Step 3 applies per-model guidance instead of a single Claude-wide default. They are the regression test for `claude-considerations.md` > "Pick the model before you tune": the failure mode is flattening the divergence, so E1 and E2 must not both resolve the same way.

| # | Context | Expected behavior | Rationale |
|---|---|---|---|
| E1 | Target runtime: Claude Opus 5. User's original prompt contains "and double-check your answer before finalizing". | The verification clause is **removed** from the final prompt, with a one-line note that it was dropped. | `claude-opus-5.md` § 2; `grounding-techniques.md` technique 3 is model-gated. Rewording it is also a failure — the documented fix is removal. |
| E2 | Target runtime: Claude Fable 5, long autonomous run. | The verifier instruction is **kept**, preferring a fresh-context verifier subagent over self-critique. | `claude-fable-5.md` § 8. Same clause, opposite call — the divergence must survive. |
| E3 | Target runtime: Claude Opus 5, quick-tier prompt with no length or tone stated. | `<output_spec>` includes an explicit length/conciseness line even at quick tier. | `claude-opus-5.md` § 1: default responses run long and `effort` does not shorten visible output, so Component 5 is load-bearing here. |
| E4 | Target runtime: Claude Opus 5, harness supports subagents. | Constraints include a delegation cap, not a delegation nudge. | `claude-opus-5.md` § 4. A "use subagents freely" line carried over from Opus 4.8 or Fable 5 guidance is a regression. |

## Fixture set F — Loadability (mechanical checks)

These fixtures protect the skill from a failure mode that has nothing to do with its logic: content that is present but never actually read. A reference file over budget can be truncated or skipped by the loading agent, and a cross-reference that points at a moved section silently degrades into no guidance at all. F1-F3 are deterministic shell checks — run them from the repository root before merging any change that adds, splits, renames, or grows a file.

| # | Check | Command | Expected |
|---|---|---|---|
| F1 | Token budgets are respected | `wc -w skills/prompt-best-practices/SKILL.md skills/prompt-best-practices/references/*.md` | `SKILL.md` ≤ 3500 words (~5000 tokens); every reference file ≤ 2700 words (~4000 tokens). |
| F2 | Every reference file declares its purpose | `rg -c '^## When to Use' skills/prompt-best-practices/references/*.md` | Every file reports `1`. A file with no count is missing the header an agent uses to decide whether loading it is worth the tokens. |
| F3 | Every path named in `SKILL.md` resolves | `rg -o 'references/[a-z0-9-]+\.md' skills/prompt-best-practices/SKILL.md \| sort -u \| while read -r p; do [ -f "skills/prompt-best-practices/$p" ] \|\| echo "MISSING: $p"; done` | No output. Reference paths in `SKILL.md` must always carry the `references/` prefix, or an agent resolving them against the working directory will fail to open the file. |
| F4 | A missing reference does not abort the workflow | Rename one reference file temporarily, then invoke the skill on a standard-tier prompt. | The skill completes Step 0 through Step 3 using `SKILL.md` alone and states in one line that a reference was unavailable. It must not decline, and it must not silently skip the diagnosis. |

## Known-limitation fixtures

Fixtures where the skill's current heuristics are weak. Document them here so contributors know what not to silently break:

- **L1**: Non-English prompts. The skill should respond in the same language, but tier classification and rubric matching may degrade. If you improve non-English detection, re-verify A1–F4 still pass.
- **L2**: Prompts with inline code blocks that contain XML. The rubric's `[OK]` detection on Component 7 (Structure) must not confuse *content XML* with *prompt-structuring XML*.

---

## Update procedure

When you add a fixture, include the rationale column — it explains *why* the outcome is correct, which prevents other contributors from "fixing" the fixture to match a regression.

When a fixture starts failing, the first move is to diagnose whether the regression is in the skill or the fixture is outdated. Do not update the fixture to match the new (broken) behavior without explicit review.
