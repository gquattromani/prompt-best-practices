# Gemini-Specific Considerations

## When to Use

Load this file when the target runtime is Google Gemini — reached from `runtime-detection.md` for Gemini CLI, Antigravity (default), Jules, or a Copilot/editor session whose picker is set to a Gemini model. It covers what differs from the Claude and OpenAI branches: sampling parameters that must be left alone, the `thinking_level` ladder, terse-by-default verbosity, delimiter choice, and the one place where Google endorses a reasoning nudge in prompt text.

## Sources

- [Prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies) — the general prompting guide (page last updated 2026-06-10), including its Gemini 3-specific section
- [Gemini 3 developer guide](https://ai.google.dev/gemini-api/docs/gemini-3) — model list, `thinking_level`, migration and prompting best practices (page dated 2026-08-18)

## Recommended models (verified 2026-08-19)

| Model | Use case |
|---|---|
| `gemini-3.1-pro-preview` | Complex tasks needing broad world knowledge and deep reasoning. The default target for a Gemini-bound prompt when nothing else is stated. |
| `gemini-3-flash-preview` | Pro-level intelligence at Flash speed and pricing — the volume workhorse. |
| `gemini-3.1-flash-lite` | Cost-efficient, high-volume, well-defined tasks. |

Image and video generation models (`gemini-3.1-flash-image-preview`, `gemini-3-pro-image-preview`) exist but are outside this framework's scope.

## The headline principles

Google states three things about Gemini 3 that shape every component:

> "Be concise in your input prompts. Gemini 3 responds best to direct, clear instructions."

> "By default, Gemini 3 is less verbose and prefers providing direct, efficient answers."

> For large datasets, "place your specific instructions or questions at the end of the prompt, after the data context."

The first aligns with the frontier-model calibration in `framework.md`. The second inverts a habit built on older models: you no longer prompt for brevity, you prompt for *expansiveness* when you want it. If the user wants a conversational tone, a walkthrough, or a long-form deliverable, Component 5 (Output specification) has to say so explicitly — the default is terse.

## Structure (Component 7)

- **Use consistent delimiters.** Google recommends XML-style tags **or** Markdown headings, and treats them as equivalent: "Employ clear delimiters to separate different parts". Pick one per prompt and do not mix.
- **Put critical instructions first, inside the System Instruction** where the harness exposes one, or at the very start of the prompt otherwise: Google's guidance is to "prioritize critical instructions".
- **Long context: data first, question last**, then anchor the instruction back to the data with a transition phrase — "Based on the information above, …". The anchor is Google-specific advice and costs one line.

The practical shape of a Gemini prompt is therefore the same skeleton as the Claude template in `framework.md`, with Markdown headings as a first-class alternative to XML tags:

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

## Sampling parameters: leave them alone

> "For all Gemini 3 models, we strongly recommend keeping the temperature parameter at its default value of `1.0`."

Lowering temperature — the reflex from earlier model generations and from other vendors — "may cause looping or degraded performance on complex tasks". The same applies to `topK`, `topP`, `max output tokens`, and `stop_sequences`: "we strongly recommend keeping them at their default values for Gemini 3.x models".

For this skill that means: **never write a temperature or sampling instruction into a Gemini-bound prompt**, and when migrating a prompt that carries one, delete it and say so in one line. This is a *delete, not a rewrite* case, the Gemini analogue of removing self-check clauses on Claude Opus 5.

## Reasoning depth: `thinking_level`, plus one text nudge

- `thinking_level` accepts `minimal`, `low`, `medium`, `high`; the default is `high` on most models. Constrain it to `low` "for faster, lower-latency responses when complex reasoning isn't required".
- It cannot be combined with the legacy `thinking_budget` in the same request.
- **Unlike Anthropic and OpenAI, Google explicitly endorses a prompt-text nudge**: "For problems that require heavy reasoning, simple requests like 'Think very hard before answering' can improve performance."

Use the parameter as the primary dial. The text nudge is a legitimate addition on genuinely hard reasoning tasks — it is the one cross-vendor divergence where a line that is wasted or harmful elsewhere earns its place here. Do not add it by default: on ordinary work it is noise, and Gemini 3 already thinks at `high`.

## Examples (Component 4)

The two Google pages pull in different directions and the resolution matters:

- The general guide is unambiguous: "We recommend to always include few-shot examples in your prompts", and explains why — few-shot examples "regulate formatting, phrasing, and response patterns".
- The Gemini 3 guide says to be concise and to "simplify prompts previously requiring chain-of-thought engineering".

Both hold once you separate the two jobs an example can do. Keep an example when it **regulates output shape** — that is the purpose Google names, and it is still the cheapest way to pin a format. Drop it when it exists to **demonstrate reasoning** or to restate a task the instruction already covers: that is the chain-of-thought scaffolding Gemini 3 no longer needs. So on Gemini, examples are not "the first thing to cut" as they are on Claude and GPT-5.6 — they are the first thing to *repurpose*, from teaching reasoning to fixing format.

## Grounding (Component 6 and `grounding-techniques.md`)

Google's answer to factual accuracy is tool enablement rather than prompt instructions:

- Grounding with Google Search "should be enabled whenever the model may need to know obscure or recent facts".
- Code execution "should be enabled whenever the model needs to perform any kind of arithmetic, counting, or calculation".

On an accuracy-critical Gemini task, therefore, prefer a constraint that names the tool ("answer only from the retrieved sources; if Search returns nothing relevant, say so") over the self-check technique in `grounding-techniques.md`. Arithmetic in particular should be delegated to code execution rather than requested carefully in prose.

## Framework mapping

| 7-component | Where it lands on Gemini 3.x |
|---|---|
| Task | Load-bearing. Direct, concise, with the success criterion stated. |
| Role | One line at most, in the System Instruction when available. |
| Context | Data first, at the top, with a transition phrase anchoring the task to it. |
| Examples | Keep for format regulation, drop for reasoning demonstration. |
| Output specification | **Heavier than on other vendors** — the default is terse, so length, depth, and tone must be requested. |
| Constraints | Rules with their reason; prefer naming the grounding tool over prescribing caution. |
| Structure | Markdown headings or XML tags, consistently; critical instructions first. |

## Migration notes

When adapting an existing prompt to Gemini 3.x:

1. Delete temperature and other sampling instructions.
2. Delete chain-of-thought scaffolding ("think step by step", staged reasoning templates); keep the "think very hard" nudge only on heavy-reasoning tasks.
3. Add an explicit length and tone line — a prompt tuned on a verbose model will produce a terser answer than its author expects.
4. Move the data above the instruction if it is not already, and add the anchor phrase.
5. For dense document work, note that `media_resolution_high` is worth testing; PDF token usage may rise while video usage typically falls.

## Update procedure

1. Re-fetch both source pages and refresh the model table and the "verified" date.
2. Re-check the three headline quotes — conciseness, terse default, instructions-last — they are the load-bearing claims here.
3. Re-verify the sampling-parameter recommendation and the `thinking_level` ladder; both are version-specific and both are the kind of thing that changes silently between generations.
4. Re-diff the divergence table in `runtime-detection.md`, especially the "think very hard" row and the examples row.

## Attribution

The passages in double quotes above are quoted from the two Google pages linked under **Sources**, reproduced under the [Creative Commons Attribution 4.0 International licence](https://creativecommons.org/licenses/by/4.0/) that covers that documentation. Attribution: Google. Quotations only — no code sample is reproduced, and an abridged quotation marks the omission with an ellipsis. Everything else in this file, including the analysis and the composition rules, is this project's own work and is covered by the repository's own licence. See `NOTICE.md`.
