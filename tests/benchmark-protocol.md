# Benchmark Protocol

## Purpose

This protocol exists to address a concrete gap: the README and this skill claim that the framework produces "significantly better results" on the first pass. That claim is testable. This file defines how to test it. Contributors running the protocol generate the empirical evidence the claim currently lacks.

The protocol is provider- and model-agnostic. The same 20 prompts are run with and without framework optimization, against the same target model, and an LLM judge scores the outputs on a rubric.

## Scope

This benchmark measures **output quality improvement from applying the framework** — not "is the skill helpful", not "do users like the dialogue". Those are different questions.

Metrics produced:
- First-pass success rate (F1-pass): fraction of outputs that meet the prompt's success criterion on the first try.
- Rubric score: 0-5 judge score on adherence, specificity, usefulness.
- Token overhead: tokens added by the framework-optimized prompt vs. baseline.
- Time-to-output: wall-clock difference (optional, measures the dialogue cost).

## Prompt set

The benchmark uses a fixed set of 20 prompts covering the three tiers (Quick: 6, Standard: 8, Complex: 6) and mixing code and content tasks. A reference set is checked in at `tests/benchmark-prompts.md` when a contributor first runs the protocol. Do not regenerate the prompt set between runs — the set is intentionally stable so results are comparable across releases.

Suggested baseline sources: the user prompts from `examples-content.md` and `examples-code.md` plus synthesized prompts covering uncovered patterns (accuracy-critical, multi-stakeholder, brand-voice-sensitive).

## Procedure

### Phase 1 — Baseline run

For each of the 20 prompts:

1. Submit the raw prompt to the target model, unmodified.
2. Record the first response verbatim.
3. Do not iterate. This is the "baseline first-pass" measurement.

### Phase 2 — Framework-optimized run

For each of the 20 prompts:

1. Run the prompt through the framework (either by invoking the skill end-to-end, or by applying `framework.md` manually with a second operator). Record the final optimized prompt.
2. Submit the optimized prompt to the same target model.
3. Record the first response verbatim.

### Phase 3 — Judging

A separate LLM (the "judge"), ideally a different model family from the target, receives for each of the 20 prompts:

- The original user prompt (for success-criterion context).
- The baseline response (A) and the framework response (B), in randomized order.
- The rubric below.

The judge returns, for each response, a 0-5 score per rubric dimension and an overall first-pass-success boolean.

### Judging rubric (0-5 each)

| Dimension | 0 | 5 |
|---|---|---|
| **Task adherence** | Ignores the task | Addresses every part of the task |
| **Success criterion fit** | No mention of success criterion | Clearly meets or exceeds success criterion |
| **Output shape** | Wrong format / length | Exact format and length requested |
| **Specificity** | Generic / boilerplate | Tailored to the user's actual context |
| **No-iteration readiness** | Requires clarification | Can be used as-is |

Report the average across dimensions and the first-pass-success rate.

## What "significantly better" means for this project

An improvement counts as significant if it clears all three of:

1. **F1-pass delta ≥ 15 percentage points** (e.g., 45% baseline → 60%+ framework).
2. **Rubric score delta ≥ 0.5 on the 0-5 scale**, averaged across dimensions.
3. **Token overhead ≤ 5× baseline prompt length**. Larger overhead means the framework is a tax, not a tool.

If fewer than three of these are met, the README claim has to be softened or rescoped.

## Reporting

Each run produces a report file at `tests/benchmark-results/<YYYY-MM-DD>-<target-model>.md` with:

- Target model and version.
- Judge model and version.
- 20 baseline / optimized / judge triplets.
- Aggregated metrics.
- Discussion of failures and outliers (these often surface rubric bugs, not framework bugs).

## Running cost

A full run costs roughly: 20 prompts × 2 conditions × target cost, plus 20 × judge cost. On current pricing, a full run against a mid-range target with a separate judge is on the order of $1-5. This is cheap enough that any non-trivial change to `framework.md` or `component-definitions.md` should include a fresh run.

## Automation

Automating the protocol is out of scope for the base repository — this is a Markdown-only skill. A contributor can add a separate harness (e.g., a Python or TypeScript runner using the provider SDK) and commit results here. If added, document the runner's entry point in `AGENTS.md`.

## Honesty clause

If a run fails to clear the "significantly better" bar, commit the results anyway. Negative evidence is more useful than silence. Update the README to match.
