# Universal Baseline (Vendor-Agnostic Prompt Composition)

## When to Use

Load this file when `runtime-detection.md` cannot resolve a vendor: a multi-model host with no visible selection (Cursor, Copilot, Windsurf, Cline, Zed, Kiro, Qoder, OpenCode, Aider, Amp, Junie, Devin, pi, Swival, OpenClaw, CodeWhale), a prompt that will be reused across runtimes, an API or CI integration, or any case where discovering the model would cost more than it is worth.

This is not a degraded mode. It is the **intersection** of what Anthropic, OpenAI, Google, and xAI each publish about their own models — every rule below is endorsed by all four, or is neutral on all four. A prompt built to this baseline is portable by construction, and a vendor file can be layered on top later without rewriting it.

## The intersection: what every vendor agrees on

1. **A specific task with an observable success criterion.** The one component no vendor lets you skip, and the one that carries the most weight everywhere.
2. **Consistent delimiters between sections, and XML tags are the safer default here.** Anthropic documents XML tags as its strongest formatting tool, Google names XML-style tags as a valid option, OpenAI has no preference, xAI does not constrain section format. The downside is asymmetric: if the hidden model turns out to be Claude, Markdown costs you Claude's strongest tool, while XML costs nothing on any of the other three. Markdown headings remain a correct alternative when the prompt will be read and edited by humans or pasted into a Markdown-heavy UI. Either way, mixing the two in one prompt is not safe.
3. **Long context first, instruction last.** Anthropic states that long inputs placed [above the query, instructions, and examples](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting) improve performance across all models; Google says to place instructions after the data context. Nothing contradicts it.
4. **Rules carry their reason.** Anthropic's advice that explaining why a behavior matters "can help Claude better understand your goals", and xAI's call for a system prompt that "describes the task, expectations, and edge-cases", are the same instruction from two directions. A motivated rule also survives paraphrase better than a bare prohibition.
5. **State each rule once.** OpenAI measures repetition as a cost and reports that repeated "ask first" lines cause needless approval requests; Anthropic reports that emphatic phrasing written for older models now overtriggers. No vendor rewards saying it twice.
6. **Positive framing.** "Do this" outperforms "don't do that" across all four; anti-patterns are a secondary list, not the primary instruction. The documented exception is narrow: when a model falls back on known defaults (frontend styles on Claude Opus 5.5, stock phrases on GPT-6), a concrete list of the patterns to avoid works where a vague one does not.
7. **One example beats a description of the format, when the format matters.** All four accept examples; only their *volume* is disputed. One is the portable number.
8. **Explicit output shape.** Format, length, and audience. Defaults differ sharply between vendors — Gemini 3.x is terse, Claude Opus 5.x runs long, GPT-6 reaches for lists and tables — so a prompt that does not state length and format gets a different answer on each. Stating it is what makes the output stable across runtimes.
9. **One authorization boundary.** Say once what the request lets the model do without asking and what needs confirmation. OpenAI asks for exactly that, Google lists it as a dimension to define for agents, Anthropic's autonomy guidance names when to pause, and xAI's edge cases cover it. Its *strength* differs by vendor — GPT-6 needs autonomy granted, Opus 5.5 needs early stops named — so the baseline states the boundary and leaves the push to the vendor file.

## The portable template

XML tags, for the reason in rule 2 — a multi-model host may well be running Claude, and no other tracked vendor penalizes them.

```xml
<role>[expertise, one line — include only if it changes the output]</role>

<context>
[documents, data, file paths — first, before the task]
</context>

<task>
[action] — success is [observable criterion].
</task>

<examples>
[one example of the desired result, only if it pins a format words cannot]
</examples>

<output_spec>
Format: [type and length]
Tone: [described positively]
Audience: [who reads it and what it should do for them]
</output_spec>

<constraints>
- [rule]: [why it exists]
- [edge case]: [what to do]
</constraints>
```

The same components in the same order with Markdown headings (`## Role`, `## Context`, …) are the equally valid alternative when a human will edit the prompt. Order is load-bearing either way: context above, instruction below. Everything else is optional and earns its place only by changing the output.

## What the baseline deliberately leaves out

Each of these is correct on at least one vendor and wrong on at least one other. Omitting them is what makes the prompt portable; add them only after the vendor is known.

| Left out | Because |
|---|---|
| Sampling values (`temperature`, `topP`, `topK`) | Google says to remove them from Gemini 3.x requests — changing them can cause looping; GPT-6 rejects them when reasoning is on; Claude Fable 5.x rejects non-default values. They are request fields, never prompt text. |
| Reasoning-depth text ("think hard", "think step by step") | Google endorses it on heavy-reasoning tasks; Anthropic and OpenAI replace it with a parameter, and Anthropic advises removing "think carefully" lines on Opus 5.5. |
| Self-check and verification clauses | Must be **removed** on Claude Opus 5 / 5.5, **kept** on Claude Fable long runs, scoped *down* on GPT-6, redundant on Gemini 3.8 Flash. A coin flip without the vendor file. |
| Prompt size posture | Anthropic, OpenAI, and Google reward lean prompts; xAI asks for a thorough system prompt with edge cases spelled out. |
| Subagent and delegation nudges | Capped on Claude Opus 5 / 5.5, encouraged on Claude Fable and GPT-6, undefined elsewhere. |
| Tool-call output envelopes ("respond with `<tool_call>`") | xAI warns that XML-based tool-call output may hurt performance versus native function calling; every vendor now has a structured tool API. |
| Assistant prefill | Unsupported on current Claude models (returns an error) and removed from Gemini 3.8 Flash requests. |
| "Show your reasoning as the answer" | A refusal trigger on Claude Fable 5.x and Opus 5.5; use the harness's thinking output instead. |
| Model names, context-window sizes, cutoff dates in prose | Any of these turns a portable prompt into a vendor-specific one, and they go stale. |

## Two rules that hold in both directions

- **Prompt for length and format explicitly.** It is the only defense against vendors whose defaults are opposite (terse on Gemini 3.x, long on Claude Opus 5.x, heavily formatted on GPT-6, lightly formatted on Claude Fable 5.1). One line in `<output_spec>` (or `## Output`).
- **Prefer the smallest prompt that removes ambiguity, but never at the cost of an edge case.** Three of four vendors reward trimming; the fourth (xAI) penalizes underspecification. Edge cases and constraints are the wrong thing to cut — examples and restated instructions are the right thing.

## Upgrading from the baseline

When the vendor becomes known mid-conversation, the prompt does not need rebuilding — apply the delta:

| Target becomes | Apply |
|---|---|
| Anthropic Claude | `claude-considerations.md`, then the per-model file. The XML structure already matches (convert if you used the Markdown variant); delete any self-check clause when the model is Opus 5 / 5.5; state response length explicitly. |
| OpenAI GPT | `codex-considerations.md`. Drop non-behavioral examples and repeated instructions; state the authorization boundary once and, on GPT-6, grant the autonomy the request implies; state the writing style; point at the `apply_patch` tool if the task edits files. |
| Google Gemini | `gemini-considerations.md`. Delete sampling instructions; add an explicit length and tone line; keep data first with an anchor phrase; add the reasoning nudge only on heavy-reasoning work. |
| xAI Grok | `grok-considerations.md`. Restore Role and expand Constraints with edge cases; enumerate context paths; describe tool use as native calling. |

## Update procedure

1. A rule stays in this file only while **every** tracked vendor endorses it or is neutral. When one vendor breaks with it, move the rule into that vendor's file and into the divergence table in `runtime-detection.md`, and remove it here.
2. When a new vendor file is added, re-check both lists above against it. A new vendor usually shrinks the intersection rather than growing it — that is the expected direction, not a regression.
3. Keep the template synchronized with `framework.md` > Final Prompt Template: same components, same order. If a vendor ever penalizes XML sections, rule 2 and both templates have to be re-decided together — and `component-definitions.md` > Component 7 > "Runtime-specific defaults" and fixture D3 in `tests/activation-fixtures.md` must move with them.
