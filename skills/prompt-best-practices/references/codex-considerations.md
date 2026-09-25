# Codex-Specific Considerations

## When to Use

Load this file when the target runtime is OpenAI Codex or another OpenAI model — the GPT-6 family (`gpt-6-astra`, `gpt-6-sol`, `gpt-6-luna`) or the previous GPT-5.6 family (`gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`). It covers behaviors that differ from Claude — the outcome-first / lean-prompt principle, the GPT-6 behavior deltas, the `reasoning.effort` ladder, verbosity control, autonomy and approval boundaries, tool use — and maps the 7-component framework onto OpenAI's current guidance.

## Sources

- [Using GPT-6](https://developers.openai.com/api/docs/guides/latest-model/gpt-6-astra) — the GPT-6 model guide, including its prompting best practices (verified 2026-09-25)
- [Prompting guidance for GPT-5.6 Sol](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6) — the lean-prompt measurement and the prompt-contract guidance the GPT-6 guide builds on
- [Reasoning models](https://developers.openai.com/api/docs/guides/reasoning) — `reasoning.effort`, reasoning modes, persisted reasoning
- [Codex / ChatGPT models](https://learn.chatgpt.com/docs/models) and [Prompting](https://learn.chatgpt.com/docs/prompting) — current model list, presets, and user-facing prompting advice

## Recommended models (verified 2026-09-25)

| Model | Use case |
|---|---|
| `gpt-6-sol` | **Everyday default in Codex** — "Use Sol for complex coding and agentic workflows". The desktop and web presets start at Sol Light. Default `reasoning.effort` is `medium`. |
| `gpt-6-astra` | Most capable — "Our most capable model, built for the hardest end-to-end work", and the API's suggested starting point. Does not accept `none` effort; Codex starts it at Light (`low`). |
| `gpt-6-luna` | "Our most efficient model for focused, high-volume tasks" — extraction, classification, structured summaries, subagents. Codex starts it at High. |
| `gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna` | Previous generation, still available during the GPT-6 rollout; Terra and Luna remain the API's cost tiers. |
| `gpt-5.5` | Retires from ChatGPT, ChatGPT Work, and Codex on 2026-10-14; stays on the API. Replace it in saved configs and scripts. |

## The headline principle: outcome-first, leaner prompts

This principle shapes how the 7-component framework applies to every OpenAI model. From the GPT-5.6 guidance:

> "Removing repeated instructions and examples and simplifying tool descriptions can improve task performance and token efficiency."

In OpenAI's internal coding-agent evals, reported in the [GPT-5.6 guidance](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6), leaner system prompts improved scores by **roughly 10–15%** while cutting total tokens by **41–66%** and cost by **33–67%** — figures OpenAI calls directional. The current user-facing guide says the same in plain words: "A short prompt is often enough", and "Use only the parts that help."

GPT-6 adds a reason to keep the prompt consistent as well as short: it "is better able to follow longer instructions, but can also be more sensitive to information in context." The GPT-5.6 page already warned that "conflicting rules can create more instability than missing detail."

**Trim:** rules stated more than once; style or process instructions that do not change behavior; examples that do not change behavior; process instructions for things the model already does reliably; tools unrelated to the task.

**Keep:** the user-visible outcome; success criteria and stopping conditions; safety, business, evidence, and permission constraints; tool-routing rules that depend on context; the required output shape.

For the 7-component framework this means: **Task** (with a real success criterion), **Output specification**, and **Constraints** (with motivation) carry almost all the weight. **Examples** belong only when one example changes the output. This is the same direction the current Claude models took (`framework.md` > "Calibrating for frontier models").

## What changes on GPT-6

The GPT-6 guide's prompts "address behavior observed with GPT-6 Astra" and are offered as a starting point across the family.

1. **It asks more, and may stop early.** The model "is thus more likely to ask the user a question when additional input could materially change the result. This can cause it to stop when the user may expect it to make reasonable assumptions and persist." When the request authorizes the work, say so, and put approval after a concrete result: `Treat a request for action as an instruction to do the work. Complete what is already authorized before asking anything, so the user approves a concrete, reviewable result; reversible and read-only actions need no permission.`

2. **Skills and rule files weigh more.** It "can be more sensitive to instructions contained in skills and other files, such as `AGENTS.md`", and OpenAI is emphatic: "We strongly recommend auditing skills and other files accessible to your model for instructions that could influence its behavior." When a prompt runs next to skills, state precedence once: `The user's instructions take precedence over a skill's guidelines when they conflict.` This matters for this skill too: its dialogue is a skill instruction that asks questions, which is why the Fast-Track Exit must always be offered.

3. **State the writing style — the default is formatted.** GPT-6 Astra "tends to use lists, tables and Markdown to make responses scannable. If your application needs prose with less formatting, specify that preference." On GPT-6, Component 5 names the format even when it seems obvious. **Note the direction:** GPT-5.6 "tends to be more concise by default than GPT-5.5"; the two generations need opposite reminders.

4. **Ask for delegation.** "The model may delegate less often than desired for your workflow. Specify when and how much it should use subagents for parallel work." **Note the divergence:** on Claude Opus 5 / 5.5 the same component carries a cap, not a nudge.

5. **Scope the testing down.** "For coding tasks, the model tends to be thorough in testing before considering a task complete." On small changes, calibrate instead of adding a verification clause: `Do not write tests for reversible, low-impact changes that mirror the implementation. Broaden or repeat testing only when failures or new changes justify it.`

6. **API changes that remove prompt-adjacent settings.** With reasoning on, `temperature`, `top_p`, and `top_logprobs` must be removed from the request. Tool calling with reasoning runs on the Responses API. To change effort between turns without breaking the cache, send a `configuration_update` item rather than editing the request-level effort.

## reasoning.effort

- `gpt-6-sol` and `gpt-6-luna`: `none`, `low`, `medium` (default), `high`, `xhigh`, `max`. `gpt-6-astra`: `low` through `max` — `none` returns a 400. The GPT-5.6 family defaults to `medium`.
- Migrating: "Preserve your current effective reasoning effort where supported", then compare one level lower — "Reasoning efforts don't map exactly between model generations."
- Before raising effort, "check whether the prompt is missing a success criterion, dependency rule, tool-routing rule, or verification loop." Reserve `max` for the hardest quality-first workloads.

## Verbosity control

Use the `text.verbosity` parameter (`low` / `medium` / `high`) for the default detail level, and specify task-specific length and structure in the prompt. For short answers the guidance recommends naming what to keep: `"Lead with the conclusion. Include the evidence needed to support it, any material caveat, and the next action."` This maps onto Component 5 (Output specification).

## Autonomy and approval boundaries

Rather than sprinkling "ask first" across the prompt, define once what each request authorizes:

> "Define what level of action each request authorizes so the model can continue safe, in-scope work without unnecessary pauses while stopping before external, destructive, costly, or scope-expanding actions."

Repetition is itself the failure: "Repeating instructions such as “ask first,” “do not mutate,” or “wait for approval” can cause unnecessary approval requests for safe, expected actions." Fold this into Component 6 (Constraints) — one boundary statement, not per-action nagging. On GPT-6 the same statement also has to *grant* autonomy (§ 1 above), not only limit it.

## Tool use

- **Parallel reads.** "When several reads are independent, parallelize them. When one result determines the next action, keep the work sequential." One Constraint line is enough.
- **Multi-agent and async tools.** GPT-6 keeps GPT-5.6's multi-agent orchestration and adds async tool calling and mid-turn steering; both are harness features, not prompt text.
- **Programmatic Tool Calling (PTC).** Best for bounded stages that reduce many tool results to a small structured output — filtering, joining, ranking, aggregation. "Multiple, parallel, or dependent calls alone do not justify Programmatic Tool Calling." Name the stage, the eligible tools, the output schema, and the retry limit.
- **`apply_patch`.** GPT-6 Sol and Astra support the Responses API `apply_patch` tool, whose `diff` field carries a V4A diff. If the Output specification references edits, point at that tool rather than asking for a freeform diff.

## Framework mapping

The 7-component framework applies to Codex through the lean-prompt lens above — include a component only when it changes behavior.

| 7-component | Where it lands on GPT-6 / GPT-5.6 |
|---|---|
| Task | Objective + success criteria + stopping condition — the load-bearing component. |
| Role | Usually implicit; a one-line function statement at most. |
| Context | Domain context and files to read; keep it to what the task needs. |
| Examples | Include only if a single example changes the output — otherwise omit. |
| Output specification | Final-message shape, verbosity, writing style (explicit on GPT-6), `apply_patch` for edits. |
| Constraints | Hard constraints, one authorization boundary, skill precedence, test scope — stated once, with motivation. |
| Structure | Markdown headings or XML both work (see below). |

## Other Codex-specific behaviors

1. **Working-tree boundaries.** OpenAI's autonomy prompt lets the model progress on its own — isolated worktrees, merge conflicts, read-only actions — "unless they are clearly destructive or irreversible". State that boundary as a Constraint when the task touches a working tree, and tell Codex about edits you reverted so it does not overwrite them.
2. **Persisted reasoning and Pro mode.** `reasoning.context` reuses reasoning across turns; `reasoning.mode: "pro"` spends more model work on hard tasks and is independent of `reasoning.effort`. Neither needs prompt text, but both reduce how much scaffolding a prompt needs.
3. **Explicit prompt caching.** Cache writes cost 1.25x the uncached input rate; keep reusable prefixes stable. Migrating from GPT-5.5 or earlier, replace `prompt_cache_retention` with `prompt_cache_options.ttl`.
4. **Structure format: Markdown headings or XML both work.** Unlike Claude, OpenAI states no preference. Its own suggested structure is a set of short labeled sections — role, personality, goal, success criteria, constraints, tools, output, stop rules — with "Add detail only where it changes behavior." See `component-definitions.md` > Component 7.
5. **Safeguards.** GPT-5.6 runs real-time cyber and biology misuse classifiers that can block or pause generation; GPT-6 Astra adds asynchronous misalignment monitoring. Send a stable, privacy-preserving `safety_identifier` for individual end-user applications.

## Update procedure

When OpenAI releases a new model or updates the prompting guidance:

1. Refresh the "Recommended models" table and the "verified" date against the models pages and the latest-model guide.
2. Re-check the lean-prompt principle — it is the load-bearing section. Record whether the new guide restates, extends, or contradicts it.
3. Re-diff "What changes on GPT-6" against the new guide: each numbered item is a behavior delta, and new generations tend to flip them.
4. Re-verify the `reasoning.effort` ladder and defaults per model, the framework-mapping table, and the tool-use section.
