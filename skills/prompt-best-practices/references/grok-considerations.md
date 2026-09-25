# Grok-Specific Considerations

## When to Use

Load this file when the target runtime is xAI Grok — reached from `runtime-detection.md` for Grok Build, or for a Copilot/editor session whose picker is set to a Grok model. It covers the one place where this framework's lean-prompt reflex is wrong: xAI asks for a **detailed** system prompt, not a minimal one. It also covers the `reasoning_effort` ladder, native tool calling versus XML tool-call output, and prompt caching.

xAI's documentation now brands the vendor "SpaceXAI" and describes itself as "Official SpaceXAI (xAI) developer documentation". This skill keeps the name xAI, the form the docs still give in parentheses.

## Sources

- [Grok 4.7 model page](https://docs.x.ai/developers/grok-4-7) — flagship model, reasoning levels, caching and long-agent-loop guidance (the former `grok-4-6` URL now serves this page)
- [Models](https://docs.x.ai/developers/models) — current lineup, context windows, and xAI's model recommendation
- [Reasoning](https://docs.x.ai/developers/model-capabilities/text/reasoning) — the `reasoning_effort` ladder
- [Grok Build overview](https://docs.x.ai/build/overview) — the coding agent, its default model, and custom-model support
- xAI's prompt-engineering guide for its coding model (`grok-code-fast-1`) — the source of the system-prompt-detail, context-specificity, and native-tool-calling recommendations quoted below. **Quotations last verified 2026-08-19**: on 2026-09-25 the guide was no longer listed in the documentation index and its known URLs returned 404. See "Known gap".

## Recommended models (verified 2026-09-25)

| Model | Use case |
|---|---|
| `grok-4.7` | **Flagship and the default model of Grok Build** (500k context). xAI's own recommendation, on the models page: "For everything else, including code, use Grok 4.7. It is the most capable model we've built." Assume this target when the runtime is Grok and nothing else is stated. |
| `grok-4.6`, `grok-4.5` | Previous flagships (500k context), still available. |
| `grok-build-0.1` | "SpaceXAI's intelligent coding model for agentic software, engineering, and workflow tasks" (256k context); `grok-code-fast-1` is now one of its aliases. |
| `grok-4.3`, `grok-4.20-*` | 1M-context models, including reasoning / non-reasoning / multi-agent variants. |

Grok Build exposes `/model <name>` in its TUI and accepts any custom model declared in `~/.grok/config.toml`, so the target can be switched — even to another vendor — inside a session. Worth one glance before assuming the default.

## The headline principle: detail earns its keep here

Every other vendor tracked by this skill asks for leaner prompts. xAI does not:

> "Be thorough and give many details in your system prompt. A well-written system prompt which describes the task, expectations, and edge-cases the model should be aware of can make a night-and-day difference."

> "Detailed and concrete queries can lead to better performance. Try to avoid vague or underspecified prompts, as they can result in suboptimal results."

This is a genuine divergence, not a wording difference, and it is the single most important thing on this page. A prompt trimmed to the bone for Gemini 3.x or Claude Opus 5.5 — role dropped, constraints compressed to one line, edge cases left implicit — is *underspecified* by xAI's own standard.

What this does **not** license: repetition, contradictory rules, or padding. "Thorough" in xAI's framing means the task, the expectations, and the edge cases are all present. The framework's answer is to keep more components rather than write more words:

- Do not cut **Role** and **Constraints** to save space, as you would on Claude Opus 5.5 or GPT-6.
- Spell out **edge cases** in Component 6 — what to do when input is missing, ambiguous, or out of scope. This is the component most often dropped by the tier caps and the one xAI names explicitly.
- Keep **Task** and **Output specification** as tight as anywhere else. Detail belongs in expectations and edge cases, not in restating the goal.

The tier caps in `SKILL.md` > Step 2 still hold — a quick-tier task does not become complex because the runtime is Grok. But within the tier, prefer the fuller component set over the leaner one.

## Context: name the files, exclude the rest

> "It is oftentimes better to be specific by selecting the specific code you want to use as context. This allows the model to focus on your task and prevent unnecessary deviations. Try to specify relevant file paths, project structures, or dependencies and avoid providing irrelevant context."

Component 3 (Context) is therefore *narrow and explicit* on Grok: enumerate the paths, the module boundaries, and the dependencies that matter, and say what is out of scope. "The codebase" is not context; `src/auth/session.rs` and `the two callers in src/api/` is.

Note the direction: more detail about *expectations*, less breadth in *context*. Those are not in tension — one specifies the job, the other specifies where to look.

## Tool calling: native, not XML

> "grok-code-fast-1 offers first-party support for native tool-calling and was specifically designed with native tool-calling in mind. We encourage you to use it instead of XML-based tool-call outputs, which may hurt performance."

This constrains one thing only, and the distinction matters for Component 7:

- **XML or Markdown delimiters for prompt sections are fine.** Nothing in xAI's guidance discourages structuring a prompt with `<task>` / `<constraints>` or `## Task` / `## Constraints`.
- **Do not ask the model to emit tool calls as XML in its response.** If the prompt's Output specification describes tool invocation, describe it as native function calling and let the harness carry the schema.

The same rule applies to any "reply in this envelope" instruction that duplicates a structured-output or tool API.

## Reasoning depth: `reasoning_effort`

| Level | xAI's description | Use for |
|---|---|---|
| `low` | "Uses some reasoning tokens, but still fast" | Latency-sensitive work |
| `medium` | "More thinking for less-latency sensitive applications" | Complex analysis |
| `high` (default) | "Uses more reasoning tokens for deeper thinking" | Challenging problems |
| `xhigh` | "Maximum reasoning depth, with correspondingly higher latency" | Quality-first work; `grok-4.6` and later only |

"Reasoning cannot be disabled." As on the other vendors, depth is a parameter and does not belong in prompt text — xAI publishes no equivalent of Google's "think very hard" endorsement, so treat a reasoning nudge as noise here.

## Caching and long agent loops

- > "We highly recommend setting a `prompt_cache_key`" — without it "you often pay full input price on a cache-cold server".
- > "Long agent loops additionally benefit from context compaction; for tool-heavy workloads see function calling."
- On the Responses API, `grok-4.7` always returns encrypted reasoning; pass the reasoning items back unchanged in the next request.

None of these is prompt text, but they shape prompt design: a structured prompt whose stable prefix (role, constraints, conventions) precedes the variable part is the cache-friendly shape, and it is what the template in `framework.md` already produces when Context and Task sit below the invariant sections. Worth one line in Component 6 only when the prompt is a reused template.

## Framework mapping

| 7-component | Where it lands on Grok 4.7 |
|---|---|
| Task | Concrete and specific; vague or underspecified prompts are called out as a failure mode. |
| Role | **Keep it.** Part of the "thorough system prompt" xAI asks for, not a component to trim. |
| Context | Narrow and enumerated — paths, structure, dependencies; exclude the irrelevant. |
| Examples | Useful, but concrete task context is worth more than generic examples. |
| Output specification | Standard; describe tool use as native calling, never as an XML envelope. |
| Constraints | **The heavy component here** — expectations and edge cases stated explicitly, each with its reason. |
| Structure | XML tags or Markdown headings both fine for sections; not for tool-call output. |

## Known gap

xAI's published prompt-engineering guidance was written for its coding model (`grok-code-fast-1`, now an alias of `grok-build-0.1`); the `grok-4.7` page documents capabilities, caching, and reasoning rather than prompt composition. The system-prompt-detail, context-specificity, and native-tool-calling recommendations above are therefore a **transfer** from that guide to the current flagship, not a vendor statement about `grok-4.7`. The gap widened on 2026-09-25: the guide itself could no longer be opened, so the quotations stand on the 2026-08-19 verification. Nothing contradicts them, and Grok Build runs `grok-4.7`, so the transfer is the best available reading — but treat it as the first item to re-verify, and replace it as soon as xAI publishes model-specific prompting guidance.

## Update procedure

1. Re-check the models table and the "verified" date against the models page, and confirm which model powers Grok Build.
2. Look for a flagship-specific prompting guide, or for the coding-model guide at a new URL. If one ships, re-verify or replace the transferred recommendations above and shrink the "Known gap" section.
3. Re-verify the `reasoning_effort` ladder (`xhigh` availability moves with the model generation) and the caching guidance.
4. Re-diff the "Prompt size" and "Examples" rows in `runtime-detection.md` — the detail-over-brevity divergence is the reason this file exists.
