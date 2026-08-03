# Claude-Specific Considerations

## When to Use

Load this file when the target runtime is Anthropic Claude. It is the **router**: it tells you which model-specific file to load, records the behaviors that hold across the current family, and flags the places where per-model tuning actively conflicts. Load it before the model-specific file — it is short by design.

| Target | File | Role |
|---|---|---|
| **Claude Opus 5** (`claude-opus-5`) | `claude-opus-5.md` | **Default target.** Complex agentic coding and enterprise work; the model most Claude agents (including Claude Code) run today. |
| **Claude Fable 5 / Mythos 5** (`claude-fable-5`, `claude-mythos-5`) | `claude-fable-5.md` | Highest-capability tier, for the hardest long-running or ambiguous work. |
| Claude Sonnet 5 (`claude-sonnet-5`) | — (see § below) | Balanced workhorse. No dedicated file yet; the Opus 5 notes transfer with the caveats listed below. |
| Claude Opus 4.8 and earlier | — (see § below) | Previous generation; still the recommended refusal-fallback target. |

If the target model is unknown, assume Claude Opus 5 — it is the current default and the most conservative choice for a Claude-bound prompt.

## How the official guidance is organized (as of 2026-08-03)

The main [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) page is organized in three parts: **model-specific guidance** first, **techniques for all current models** after (general principles, output and formatting, tool use, thinking, agentic systems), and **migration considerations** last. Model-specific behavior lives on four dedicated pages:

- [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) — response verbosity, agentic narration, task scoping, subagent delegation, self-correction, thinking-disabled artifacts.
- [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — effort, instruction following, long runs, memory, scaffolding changes, the `reasoning_extraction` refusal category. Covers Claude Mythos 5.
- [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5) — response length, effort and thinking-depth calibration, tool-use triggering, literal instruction following, design defaults.
- [Prompting Claude Opus 4.8](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8) — the previous generation.

When a newer flagship ships, follow the [update procedure](#update-procedure).

## Pick the model before you tune

Per-model guidance is not additive: several instructions that help one model measurably hurt another. Applying the wrong column is not neutral.

| Behavior | Claude Opus 5 | Claude Fable 5 / Mythos 5 |
|---|---|---|
| Self-check / verification instruction | **Remove it.** The model verifies on its own; the instruction causes over-verification. | **Keep it** on long runs — fresh-context verifier subagents outperform self-critique. |
| Subagent delegation | **Cap it.** Delegates readily; explicit criteria or a hard cap. | **Encourage it.** Dispatches parallel subagents dependably. |
| Response length | **Must be prompted.** `effort` changes thinking volume, not visible length. | A one-line brevity instruction is enough. |
| Correction narration | Needs damping in user-facing products. | Not a reported issue. |
| Thinking config | On by default; `disabled` accepted only at `effort` ≤ `high`. | Always on; no disable, no `budget_tokens`. |
| Effort posture | Default `high`; `low`/`medium` are the primary cost lever; `xhigh` for demanding coding/agentic work. | Default `high`; `xhigh` for the most capability-sensitive work; `medium`/`low` for routine. |
| Refusal fallback | Configure one (`fallbacks: "default"` routes by category). | Configure fallback to Claude Opus 4.8. |

The two most expensive mistakes: carrying a "verify before finalizing" clause into an Opus 5 prompt, and carrying a "delegate to subagents" nudge written for Opus 4.8 into an Opus 5 prompt. Both add cost with no quality gain.

## Behaviors shared across the current family

1. **XML tags remain the strongest formatting tool on Claude.** Claude parses XML tags unambiguously; use them consistently for any prompt with more than two components. This is Claude-specific — `component-definitions.md` > Component 7 lists the runtime-agnostic options.
2. **Prefilled assistant responses are unsupported.** From Claude 4.6 onward, prefilling the last assistant turn returns a 400. Use system-prompt instructions, Structured Outputs, XML output tags, or a user-turn continuation instead.
3. **Adaptive thinking plus `effort` replaced thinking budgets.** `budget_tokens` returns a 400 on Claude 4.7 and later. Depth of reasoning is a parameter, not prompt text — do not write "think hard" or "be thorough" in place of setting `effort`.
4. **Motivation beats bare rules.** "Claude is smart enough to generalize from the explanation" — the basis of Component 6. `NEVER use ellipses` underperforms `never use ellipses: the response will be read aloud by a text-to-speech engine`.
5. **Do not ask Claude to reproduce its internal reasoning as response text.** It is a refusal trigger on Fable 5 (`reasoning_extraction`) and pointless elsewhere — read the structured `thinking` blocks instead, or surface progress through a tool.
6. **Instructions are followed literally.** Across the family, aggressive framing (`CRITICAL:`, `MUST`, `If in doubt, use X`) overtriggers, and conservative framing ("only report high-severity issues") under-triggers. State the rule once, at the strength you actually want.

## Claude Sonnet 5

No dedicated file yet. The Opus 5 notes are the closest fit, with two known differences from its [own page](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5): Sonnet 5 needs explicit tool-use triggering in some workloads (Opus 5 does not), and it interprets instructions more literally at low effort, so scope should be stated explicitly. Verify against the page before relying on the transfer.

## Claude Opus 4.8 and earlier

Opus 4.8 remains widely deployed and is the recommended refusal-fallback target for Fable 5. Its dedicated page still applies. The load-bearing differences from Opus 5: it interprets instructions more literally at low effort (state scope explicitly), starts coding and agentic work at `xhigh`, **under**-reaches for subagents and file-based memory (so it needs the nudge Opus 5 needs capped), and does not have the verbosity or over-verification behaviors described in `claude-opus-5.md`. When a prompt has to stay valid for a model-plus-fallback pipeline, keep it to the intersection — the 7-component framework is, by construction.

## Update procedure

When Anthropic releases a new flagship or updates a per-model page:

1. Check whether the model gets its own `prompting-claude-<model>` page (the current pattern) or in-page guidance.
2. Add or promote the model file (`claude-<model>.md`) and update the routing table above. Demote the outgoing default to the "previous generation" section if it is still the fallback target.
3. **Re-diff the "Pick the model before you tune" table.** New generations tend to invert prior tuning rather than extend it (Opus 4.8 needed subagent encouragement; Opus 5 needs a cap). A row that flips is the highest-value thing to record.
4. Update the "How the official guidance is organized" block and the verification record in `maintenance.md`.
5. Re-check the over-prescription directive against the new page — frontier models keep raising default performance, which usually means *fewer* framework components add value, not more.
6. Re-run `tests/activation-fixtures.md` (fixture set D covers runtime-specific behavior).
