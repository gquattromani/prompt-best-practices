# Activation Regression Fixtures

## Purpose

Reference prompts with expected activation outcomes. A contributor modifying `SKILL.md` or `framework.md` must verify that every fixture still produces its expected outcome. This is the regression test for the skill's behavior.

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
| D1 | Target runtime: Claude (Opus 4.8, Claude Code) | XML tags | `framework.md` > Component 7 default; `claude-considerations.md` §8. |
| D2 | Target runtime: Codex (`gpt-5.5`) | Markdown headings or XML, both acceptable | `codex-considerations.md` §7. |
| D3 | Target runtime not specified | XML tags (safe default) | `framework.md` > Component 7 > Runtime-specific defaults. |

## Known-limitation fixtures

Fixtures where the skill's current heuristics are weak. Document them here so contributors know what not to silently break:

- **L1**: Non-English prompts. The skill should respond in the same language, but tier classification and rubric matching may degrade. If you improve non-English detection, re-verify A1–D3 still pass.
- **L2**: Prompts with inline code blocks that contain XML. The rubric's `[OK]` detection on Component 7 (Structure) must not confuse *content XML* with *prompt-structuring XML*.

---

## Update procedure

When you add a fixture, include the rationale column — it explains *why* the outcome is correct, which prevents other contributors from "fixing" the fixture to match a regression.

When a fixture starts failing, the first move is to diagnose whether the regression is in the skill or the fixture is outdated. Do not update the fixture to match the new (broken) behavior without explicit review.
