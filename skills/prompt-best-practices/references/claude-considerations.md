# Claude-Specific Considerations

## When to Use

Load this file when the target runtime is Anthropic Claude — normally reached from `runtime-detection.md`, which resolves the host to a vendor first. It is the **second-level router** for the Anthropic branch: it tells you which model-specific file to load, records the behaviors that hold across the current family, and flags the places where per-model tuning actively conflicts. Load it before the model-specific file — it is short by design.

| Target | File | Role |
|---|---|---|
| **Claude Opus 5.5** (`claude-opus-5-5`) | `claude-opus-5-5.md` | **Default target.** Long-running agentic coding and knowledge work; the default model in Claude Code on paid plans and on the API. |
| **Claude Fable 5.1 / Mythos 5.1** (`claude-fable-5-1`, `claude-mythos-5-1`), and Fable 5 / Mythos 5 | `claude-fable-5.md` | Highest-capability tier, for the hardest long-running or ambiguous work. |
| Claude Opus 5 (`claude-opus-5`) | `claude-opus-5-5.md` + § below | Previous default. Its tuning points are the base of the Opus 5.5 file; the § below lists what differs. |
| Claude Sonnet 5 (`claude-sonnet-5`) | — (see § below) | Balanced workhorse. No dedicated file; the Opus notes transfer with the caveats listed below. |
| Claude Opus 4.8 and earlier | — (see § below) | Previous generation; one of the two permitted fallback targets for Fable 5.1. |

If the target model is unknown, assume Claude Opus 5.5 — it is the current default and the model Anthropic's own sample prompts name.

## How the official guidance is organized (as of 2026-09-25)

The main [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) page is organized in three parts: **model-specific guidance** first, **techniques for all current models** after (general principles, output and formatting, tool use, thinking, agentic systems), and **migration considerations** last. Model-specific behavior lives on six dedicated pages, each written as a delta from its predecessor:

- [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) — effort, finishing long tasks, progress updates, append-only history, tool-call batching, search at low effort, formatting, writing density. Covers Mythos 5.1.
- [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — effort, instruction following, long runs, memory, the `reasoning_extraction` refusal category. Covers Mythos 5.
- [Prompting Claude Opus 5.5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) — effort calibration, prompts written for thinking disabled, progress updates, unattended runs, safeguard refusals, visual inputs.
- [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) — verbosity, narration, task scope and over-verification, subagent control, self-correction, thinking disabled.
- [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5) — response length, effort and thinking depth, tool-use triggering, literal instruction following.
- [Prompting Claude Opus 4.8](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8) — the previous generation.

When a newer flagship ships, follow the [update procedure](#update-procedure).

## Pick the model before you tune

Per-model guidance is not additive: several instructions that help one model measurably hurt another. The general guide says so outright: "Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another."

| Behavior | Claude Opus 5.5 | Claude Opus 5 | Claude Fable 5.1 (Fable 5 where different) |
|---|---|---|---|
| Self-check / verification instruction | **Remove it** — inherited from Opus 5. | **Remove it.** The instruction causes over-verification. | **Keep it** on long runs — fresh-context verifier subagents outperform self-critique. |
| Subagent delegation | **Cap it** (inherited); a time budget speeds up agent teams. | **Cap it.** Delegates readily. | **Encourage it**; let the lead keep working while subagents run. |
| Response length and narration | Length must be prompted; describe the update cadence you want. | Length must be prompted; narration needs damping. | **Ask for progress updates** and delete "hold findings" lines (Fable 5: a one-line brevity instruction is enough). |
| Formatting rules | Not a reported issue. | Not a reported issue. | **Delete anti-formatting blocks** — it already formats less. |
| Thinking config | Always on; `disabled` returns a 400; drop "think carefully" lines from chat prompts. | On by default; `disabled` accepted only at `effort` ≤ `high`. | Always on; no disable, no `budget_tokens`. |
| Effort default | `medium` | `high` | `high`; `max` available on 5.1. |
| Refusal fallback | `fallbacks: "default"`; `reasoning_extraction` declines are returned, not retried. | `fallbacks: "default"` routes by category. | Permitted targets: Claude Opus 4.8 and Claude Opus 5. |

The four most expensive mistakes: carrying a "verify before finalizing" clause into an Opus 5 / 5.5 prompt; carrying a "delegate to subagents" nudge written for Opus 4.8 into an Opus 5 / 5.5 prompt; carrying an anti-formatting block or a "hold all findings for the final response" line into a Fable 5.1 prompt; carrying a thinking-disabled configuration, or an instruction to write the reasoning into the response, into an Opus 5.5 prompt. Each adds cost or triggers a failure with no quality gain.

## Behaviors shared across the current family

1. **XML tags remain the strongest formatting tool on Claude.** Claude parses XML tags unambiguously; use them consistently for any prompt with more than two components. This is Claude-specific — `component-definitions.md` > Component 7 lists the runtime-agnostic options.
2. **Prefilled assistant responses are unsupported.** From Claude 4.6 onward, prefilling the last assistant turn returns a 400. Use system-prompt instructions, Structured Outputs, XML output tags, or a user-turn continuation instead.
3. **Adaptive thinking plus `effort` replaced thinking budgets.** `budget_tokens` returns a 400 on Claude 4.7 and later, and thinking is always on for Fable 5.x and Opus 5.5. Depth of reasoning is a parameter, not prompt text — do not write "think hard" or "be thorough" in place of setting `effort`.
4. **Motivation beats bare rules.** "Claude is smart enough to generalize from the explanation" — the basis of Component 6. `NEVER use ellipses` underperforms `never use ellipses: the response will be read aloud by a text-to-speech engine`.
5. **Do not ask Claude to reproduce its internal reasoning as response text.** It is a refusal trigger (`reasoning_extraction`) on Fable 5.x and Opus 5.5, and pointless elsewhere — read the structured `thinking` blocks instead, or surface progress through a tool.
6. **Instructions are followed literally.** Aggressive framing (`CRITICAL:`, `MUST`, `If in doubt, use X`) overtriggers, and conservative framing ("only report high-severity issues") under-triggers. State the rule once, at the strength you actually want.
7. **Forced tool use is gone on the newest models.** `tool_choice` of `any` or a named tool returns a 400 on Opus 5.5 and Fable 5.1. Describe when a tool applies in the prompt instead of relying on the API to force it; per-turn reminders go in turn-scoped system messages rather than edits to earlier turns, because thinking blocks are bound to the conversation that produced them.

## Claude Opus 5

The previous default, still widely deployed. Every point in `claude-opus-5-5.md` > "Inherited from Opus 5" applies to it as measured. The differences from Opus 5.5: the default `effort` is `high`; it has no biology or `reasoning_extraction` classifier; it "narrates readily during agentic work", so describe a lean cadence (one sentence before the first tool call, updates only on a real finding, outcome first at the end); and thinking can be disabled at `effort` `high` or below. With thinking disabled two artifacts can appear — a tool call written into the visible text, and internal `<thinking>` tags in the response. The primary fix is to keep thinking on at `low` effort; if it must stay off, remove any rule telling the model not to think and add one combined instruction: `You may say a brief sentence before a tool call. If no tool fits, say so instead of guessing. Do not include internal or system XML tags in the response.`

## Claude Sonnet 5

No dedicated file. The Opus notes are the closest fit, with differences from its [own page](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5): with thinking disabled it is less likely to reach for tools, so a tool-reliant prompt needs an explicit nudge; it "interprets prompts literally and explicitly, particularly at lower effort levels", so scope must be stated ("every section, not just the first one"); and its progress updates are good by default, so scaffolding that forces interim status messages can go. Verify against the page before relying on the transfer.

## Claude Opus 4.8 and earlier

Opus 4.8 remains widely deployed and is, with Opus 5, a permitted refusal-fallback target for Fable 5.1. Its dedicated page still applies. The load-bearing differences from the Opus 5 generation: it interprets instructions more literally at low effort (state scope explicitly), starts coding and agentic work at `xhigh`, **under**-reaches for subagents and file-based memory (so it needs the nudge Opus 5 needs capped), and does not have the verbosity or over-verification behaviors described in `claude-opus-5-5.md`. When a prompt has to stay valid for a model-plus-fallback pipeline, keep it to the intersection — the 7-component framework is, by construction.

## Update procedure

When Anthropic releases a new flagship or updates a per-model page:

1. Check whether the model gets its own `prompting-claude-<model>` page (the current pattern) or in-page guidance.
2. Decide *file or delta*: if the new page inverts a row of the "Pick the model before you tune" table, give the model its own `claude-<model>.md` and demote the outgoing model to a section here (the Opus 5 → 5.5 case); if it only adds behavior on top of an unchanged table, extend the existing file (the Fable 5 → 5.1 case).
3. **Re-diff the "Pick the model before you tune" table.** New generations tend to invert prior tuning rather than extend it (Opus 4.8 needed subagent encouragement, Opus 5 needs a cap; earlier models overused formatting, Fable 5.1 underuses it). A row that flips is the highest-value thing to record.
4. Update the routing table, the "How the official guidance is organized" block, and the verification record in `maintenance.md`.
5. Re-check the over-prescription directive against the new page — frontier models keep raising default performance, which usually means *fewer* framework components add value, not more.
6. Re-run `tests/activation-fixtures.md` (fixture sets D and E cover runtime- and model-specific behavior).
