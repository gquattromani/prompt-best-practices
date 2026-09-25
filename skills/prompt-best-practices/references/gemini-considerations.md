# Gemini-Specific Considerations

## When to Use

Load this file when the target runtime is Google Gemini — reached from `runtime-detection.md` for Gemini CLI, Antigravity (default), Jules, or a Copilot/editor session whose picker is set to a Gemini model. It covers what differs from the Claude and OpenAI branches: sampling parameters that must be removed, the `thinking_level` ladder, terse-by-default verbosity, delimiter choice, and the one place where Google endorses a reasoning nudge in prompt text.

## Sources

- [Prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies) — the general prompting guide (page last updated 2026-09-17), including its Gemini 3 section
- [Gemini 3.8 Flash](https://ai.google.dev/gemini-api/docs/latest-model) — the current model guide (page last updated 2026-09-23)
- [What's new in Gemini 3.5 Flash](https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5) — the parameter updates and prompting best practices that apply to all Gemini 3.x models (page last updated 2026-09-23). It replaces the Gemini 3 developer guide, which now carries a deprecation notice.

## Recommended models (verified 2026-09-25)

| Model | Use case |
|---|---|
| `gemini-3.8-flash` | GA; "our most intelligent Flash model, engineered for long-horizon software engineering, autonomous agents, and complex enterprise workflows". Default `thinking_level` `medium`; `minimal` returns an error. The model behind the Antigravity agent and SDK. **The default target for a Gemini-bound prompt when nothing else is stated.** |
| `gemini-3.1-pro-preview` | Preview; complex problem-solving and deep reasoning. Default `thinking_level` `high`. |
| `gemini-3.7-flash`, `gemini-3.6-flash` | Previous-generation Flash models, still fully supported. |
| `gemini-3.5-flash-lite`, `gemini-3.1-flash-lite` | Cost-efficient, high-volume, well-defined tasks. |

Image, video, speech, and Live models exist but are outside this framework's scope.

## The headline principles

Google states three things about Gemini 3.x that shape every component:

> "Be concise. Gemini 3.x responds best to direct, clear instructions. Verbose or complex prompt engineering techniques designed for older models may cause the model to over-analyze."

> "By default, Gemini 3 models provide direct and efficient answers. If you need a more conversational or detailed response, you must explicitly request it in your instructions."

> For large datasets, "place your specific instructions or questions at the end of the prompt, after the data context."

The first aligns with the frontier-model calibration in `framework.md`. The second inverts a habit built on older models: you no longer prompt for brevity, you prompt for *expansiveness* when you want it. If the user wants a conversational tone, a walkthrough, or a long-form deliverable, Component 5 (Output specification) has to say so explicitly — the default is terse.

Gemini 3.8 Flash adds one behavior that moves a component: it "takes smaller reasoning steps, calls tools iteratively, and verifies its work along the way", which is why it can use more tokens on long tasks. Google's fix for workflows that do not need that much checking is a lower thinking level, not a prompt clause — so a self-check instruction duplicates default behavior here.

## Structure (Component 7)

- **Use consistent delimiters.** Google treats XML-style tags and Markdown headings as equivalent: "Employ clear delimiters to separate different parts of your prompt." Pick one per prompt and do not mix.
- **Put critical instructions first.** "Place essential behavioral constraints, role definitions (persona), and output format requirements in the System Instruction or at the very beginning of the user prompt."
- **Long context: data first, question last**, then anchor the instruction back to the data with a transition phrase — "Based on the information above..." in the general guide, "Based on the preceding information..." in the 3.x guide. The anchor is Google-specific advice and costs one line.

The practical shape of a Gemini prompt is the same skeleton as the Claude template in `framework.md`, with Markdown headings as a first-class alternative to XML tags:

```markdown
## Role
[persona, one line, only if it changes behavior]

## Context
[documents and data — first, before the task]

## Task
[action] — success is [criterion].

## Output
Format: [type and length — state it, the default is terse]
Tone: [only if you want something other than direct and efficient]

## Constraints
- [rule]: [why]
```

## Sampling parameters: remove them

The guidance hardened between releases. `temperature`, `top_p`, and `top_k` "are no longer recommended for all Gemini 3.x models", and the instruction is direct: "Remove these parameters from all requests." The general guide gives the reason — changing them "(for example, setting the temperature below 1.0) can cause unexpected behavior, such as looping or degraded performance, particularly in complex mathematical or reasoning tasks."

For this skill that means: **never write a temperature or sampling instruction into a Gemini-bound prompt**, and when migrating a prompt that carries one, delete it and say so in one line. This is a *delete, not a rewrite* case, the Gemini analogue of removing self-check clauses on Claude Opus 5 / 5.5. Where the user wanted low temperature for repeatability, Google's replacement belongs in Constraints: "To ensure determinism, we recommend defining a system instruction with explicit rules for your specific use case."

## Reasoning depth: `thinking_level`, plus one text nudge

- `thinking_level` accepts `minimal`, `low`, `medium`, `high`. The default moved to `medium` on Gemini 3.5 Flash and 3.8 Flash ("changed from `high`"); Gemini 3.1 Pro still defaults to `high`; 3.8 Flash does not accept `minimal`.
- It cannot be combined with the legacy `thinking_budget` in the same request.
- **Unlike Anthropic and OpenAI, Google explicitly endorses a prompt-text nudge**: "For problems that require heavy reasoning, simple requests like 'Think very hard before answering' can improve performance, though at the cost of extra thinking tokens."

Use the parameter as the primary dial. The text nudge is a legitimate addition on genuinely hard reasoning tasks — the one cross-vendor divergence where a line that is wasted or harmful elsewhere earns its place here. Do not add it by default: on ordinary work it is noise, and raising `thinking_level` is the documented first move.

## Examples (Component 4)

The Google pages pull in different directions and the resolution matters:

- The general guide is unambiguous: "We recommend to always include few-shot examples in your prompts", because few-shot prompts "are often used to regulate the formatting, phrasing, scoping, or general patterning of model responses."
- The 3.x guide says to simplify: "If you used chain-of-thought prompt engineering to force reasoning, try `thinking_level: "medium"` or `"high"` with simpler prompts instead."

Both hold once you separate the two jobs an example can do. Keep an example when it **regulates output shape** — the purpose Google names, and still the cheapest way to pin a format. Drop it when it exists to **demonstrate reasoning** or to restate a task the instruction already covers: that is the chain-of-thought scaffolding Gemini 3.x no longer needs. So on Gemini, examples are not "the first thing to cut" as they are on Claude and GPT — they are the first thing to *repurpose*, from teaching reasoning to fixing format.

## Grounding and tool use (Component 6 and `grounding-techniques.md`)

Google's answer to factual accuracy is tool enablement rather than prompt instructions:

- Grounding with Google Search "should be enabled whenever the model may need to know obscure or recent facts".
- Code execution "should be enabled whenever the model needs to perform any kind of arithmetic, counting, or calculation".

On an accuracy-critical Gemini task, therefore, prefer a constraint that names the tool ("answer only from the retrieved sources; if Search returns nothing relevant, say so") over the self-check technique in `grounding-techniques.md`. Arithmetic in particular should be delegated to code execution rather than requested carefully in prose.

If an agent over-uses tools, lower the thinking level first; if that is not enough, a budget line in the system instruction works: "You have a limited action budget of <n> tool calls. Use them efficiently." For agentic prompts, the general guide also names what to define explicitly, including when the model "is permitted to make assumptions versus when it must pause execution to ask the user for clarification or permission."

## Framework mapping

| 7-component | Where it lands on Gemini 3.x |
|---|---|
| Task | Load-bearing. Direct, concise, with the success criterion stated. |
| Role | One line at most, in the System Instruction when available. |
| Context | Data first, at the top, with a transition phrase anchoring the task to it. |
| Examples | Keep for format regulation, drop for reasoning demonstration. |
| Output specification | **Heavier than on other vendors** — the default is terse, so length, depth, and tone must be requested. |
| Constraints | Rules with their reason; name the grounding tool instead of prescribing caution; state the assumption-versus-ask boundary on agentic work. |
| Structure | Markdown headings or XML tags, consistently; critical instructions first. |

## Migration notes

When adapting an existing prompt to Gemini 3.x:

1. Delete temperature and other sampling instructions; move any determinism requirement into explicit rules.
2. Delete chain-of-thought scaffolding ("think step by step", staged reasoning templates); keep the "think very hard" nudge only on heavy-reasoning tasks.
3. Add an explicit length and tone line — a prompt tuned on a verbose model will produce a terser answer than its author expects.
4. Move the data above the instruction if it is not already, and add the anchor phrase.
5. Remove prefilled model turns: the 3.8 Flash migration checklist lists them among the turn-validation rules to enforce.
6. Re-test quality at the new `medium` default before raising `thinking_level`.

## Update procedure

1. Re-fetch the three source pages and refresh the model table and the "verified" date. The model guide URL is `latest-model` and moves with each release — check which model it now describes.
2. Re-check the three headline quotes — conciseness, terse default, instructions-last — they are the load-bearing claims here.
3. Re-verify the sampling-parameter guidance and the `thinking_level` ladder and per-model defaults; both have changed between 3.x releases without a major version bump.
4. Re-diff the divergence table in `runtime-detection.md`, especially the "think very hard", sampling, and examples rows.

## Attribution

The passages in double quotes above are quoted from the Google pages linked under **Sources**, reproduced under the [Creative Commons Attribution 4.0 International licence](https://creativecommons.org/licenses/by/4.0/) that covers that documentation (footer confirmed on all three pages, 2026-09-25). Attribution: Google. Quotations only — no code sample is reproduced, and an abridged quotation marks the omission with an ellipsis. Everything else in this file, including the analysis and the composition rules, is this project's own work and is covered by the repository's own licence. See `NOTICE.md`.
