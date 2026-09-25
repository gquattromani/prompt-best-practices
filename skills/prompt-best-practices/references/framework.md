# Prompt Structuring Framework

## When to Use

This is the canonical entry point for the framework. Load it whenever you need the task-tier mapping, the frontier-model calibration, the prompt-assembly template, or the provenance of a component in Anthropic's guide.

It is deliberately small enough to load in full. Everything else lives in a sibling file, one topic each:

| File | Load when you need | Facing |
|---|---|---|
| `component-definitions.md` | the formal definition, format, and check-list of a component (1-7) | runtime |
| `component-rubrics.md` | to decide `[OK]` / `[~~]` / `[--]` for a component deterministically | runtime |
| `runtime-detection.md` | to map the host you are running in to the model family that will consume the prompt, plus the cross-vendor divergence table | runtime |
| `universal-baseline.md` | the vendor is unresolved — the portable intersection of all tracked vendors | runtime |
| `claude-considerations.md` | the Claude router: which per-model file applies, family-wide behaviors, divergence table | runtime |
| `claude-opus-5-5.md` | tuning for Claude Opus 5.5 — the default target when no model is stated — and Claude Opus 5 | runtime |
| `claude-fable-5.md` | tuning for Claude Fable 5.1 / Mythos 5.1 and Fable 5 / Mythos 5 | runtime |
| `codex-considerations.md` | tuning for OpenAI Codex (GPT-6 and GPT-5.6 families) | runtime |
| `gemini-considerations.md` | tuning for Google Gemini (Gemini 3.x) | runtime |
| `grok-considerations.md` | tuning for xAI Grok (`grok-4.7`) | runtime |
| `grounding-techniques.md` | accuracy techniques for accuracy-critical tasks (technique 3 is model-gated) | runtime |
| `examples-code.md` / `examples-content.md` | a worked end-to-end dialogue example | runtime |
| `maintenance.md` | the verification record and the update procedure for new guide revisions | contributor |

Paths are relative to this folder. Every file is self-contained and opens with its own `## When to Use`.

---

## Overview

This framework defines 7 components for building effective prompts. Each component is derived from a specific section of [Anthropic's official prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices).

The framework is **modular, not prescriptive**. The best approach depends on the type of task — not every prompt needs all 7 components. Use what adds value and skip what does not.

### The 7 components at a glance

Enough to run the Step 0 assessment and the Step 1 diagnosis without loading anything else. For the full definition of any row — format, what to check for, present/missing criteria — load `component-definitions.md`.

| # | Component | One-line definition | Tag |
|---|---|---|---|
| 1 | Task | A specific action with measurable or observable success criteria. | `<task>` |
| 2 | Role | A persona or expertise area that focuses the model's behavior. | `<role>` |
| 3 | Context | Documents, data, or background — structured, and placed at the top. | `<documents>` |
| 4 | Examples | Concrete samples of the desired output. First component to cut on Claude and GPT; on Gemini keep one when it pins a format. | `<examples>` |
| 5 | Output specification | Format, length, tone, audience impact, success metric — positive framing first. | `<output_spec>` |
| 6 | Constraints | Explicit rules, each with the reason it exists. | `<constraints>` |
| 7 | Structure | Consistent delimiters separating the sections. Format-agnostic; XML on Claude, Markdown elsewhere or when the vendor is unknown. | — |

### Source mapping

Each component links to the section of the official guide it derives from.

| # | Component | Source section | Key quote |
|---|---|---|---|
| 1 | Task | [General principles > Be clear and direct](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#be-clear-and-direct) | "Being specific about your desired output can help enhance results." |
| 2 | Role | [General principles > Give Claude a role](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#give-claude-a-role) | "Setting a role in the system prompt focuses Claude's behavior and tone for your use case." |
| 3 | Context | [General principles > Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting) | "Put longform data at the top: Place your long documents and inputs near the top of your prompt, above your query, instructions, and examples." |
| 4 | Examples | [General principles > Use examples effectively](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#use-examples-effectively) | "A few well-crafted examples … improve accuracy and consistency." |
| 5 | Output specification | [Output and formatting > Control the format of responses](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#control-the-format-of-responses) | "Tell Claude what to do instead of what not to do." |
| 6 | Constraints | [General principles > Add context to improve performance](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#add-context-to-improve-performance) | "Providing context or motivation behind your instructions … can help Claude better understand your goals …" |
| 7 | Structure | [General principles > Structure prompts with XML tags](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#structure-prompts-with-xml-tags) | "XML tags help Claude parse complex prompts unambiguously …" |

Sources for the secondary reference files are listed in `maintenance.md` > Additional source mapping.

### Task Tiers

"Modular, not prescriptive" is operationalized through three task tiers. Each tier sets the *required* component count and the *dialogue cap* used by `SKILL.md` > Step 2. A prompt meeting or exceeding its tier threshold does not need auto-activation; slash-command invocation still assesses but fast-tracks quickly.

| Tier | Scope signal | Required components | Dialogue cap | Typical template |
|---|---|---|---|---|
| **Quick** | Trivial artifact, ~1-3 min of human work. Examples: commit message, single variable rename request, one-line doc comment. | 2 (Task + Output spec) | 2 questions | `<task>` + `<output_spec>` |
| **Standard** | Bounded artifact with stakes or conventions, ~15-60 min of human work. Examples: PR description, a REST route, a marketing email. | 3-4 (Task + Output spec + Examples or Constraints) | 3 questions | adds `<examples>` or `<constraints>` |
| **Complex** | Open-ended artifact, multiple stakeholders, accuracy-critical constraints, or external context, >1 hr of human work. Examples: auth rewrite, migration, cross-team announcement. | 5+ (all that add value) | 5 questions | full template |

**Signals for tier detection:**

- **Quick** — the artifact fits in a single paragraph or function; correctness is self-evident; no external context is needed.
- **Standard** — the task references conventions ("follow our style", "match the existing pattern") or has an audience with expectations; getting it right on the first pass reduces iteration.
- **Complex** — the task mentions multiple subsystems, compliance/security, external dependencies, or requires reading existing code or documents before writing.

Prefer the lower tier when in doubt. A well-aimed 2-component prompt beats a padded 7-component one.

---

## Calibrating for frontier models

Three of the four tracked vendors — Anthropic (Claude Opus 5.5, Claude Fable 5.1 / Mythos 5.1), OpenAI (GPT-6, with the lean-prompt measurement published for GPT-5.6), and Google (Gemini 3.x) — converge on the same directive, and it sharpens how this framework should be used. **xAI is the exception and it is a load-bearing one**: for Grok, xAI asks for a thorough system prompt that names expectations and edge cases, so the trimming rules below are the wrong reflex on that runtime (`grok-considerations.md` > headline; the divergence table lives in `runtime-detection.md`).

- **Claude Fable 5:** "Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality. Review and consider removing older instructions if default performance is better." Fable 5.1 keeps the posture: existing Fable 5 prompts "should perform well on Claude Fable 5.1 without changes". (`claude-fable-5.md` > headline)
- **Claude Opus 5 / 5.5:** verification instructions "cause over-verification on Claude Opus 5, and removing them reduces wasted tokens with no loss in quality" — and the official instruction is to *remove* them "rather than rewriting them". Opus 5.5 inherits the Opus 5 patterns as its starting point and adds a second delete: "think carefully" lines in chat system prompts. (`claude-opus-5-5.md`)
- **OpenAI GPT-5.6 / GPT-6:** "Removing repeated instructions and examples and simplifying tool descriptions can improve task performance and token efficiency." Leaner system prompts scored roughly 10–15% higher while using 41–66% fewer tokens ([OpenAI, GPT-5.6 guidance](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6)); GPT-6 adds that it "can also be more sensitive to information in context", which makes a stale or conflicting line more expensive. (`codex-considerations.md` > "outcome-first, leaner prompts")
- **Google Gemini 3.x:** "Be concise. Gemini 3.x responds best to direct, clear instructions." Its verbosity default runs the other way, though — "By default, Gemini 3 models provide direct and efficient answers", so length and tone have to be requested rather than curtailed. (`gemini-considerations.md` > headline principles)

The framework's job is to make the *intent* precise — a clear task, a real success criterion, motivated constraints, the right output shape — **not to maximize component count.** On these models:

1. **A component earns its place only if it changes the output.** If the model would produce the same result without a section, drop the section. This is the "modular, not prescriptive" rule made literal.
2. **Examples are the first thing to cut — the redundant ones.** Frontier models need far fewer few-shot examples; include one only when it demonstrably steers format, tone, or a behavior that words cannot. Anthropic's general guide still recommends "3–5 examples for best results" when examples are the steering tool, and Fable 5.1's documented fix for unmarked quotations is one complete example, so the rule is "cut the example that restates the task", not "cut every example".
3. **State a rule once, with its reason.** Repetition and per-action nagging ("ask first", "don't forget to…") degrade rather than help. Component 6's motivation clause replaces enumeration.
4. **Prefer a brief steering instruction over an enumeration.** "Lead with the outcome; keep it concise" outperforms a bulleted list of ten things to avoid.
5. **Effort is a dial for *depth*, not for *length*.** Reasoning depth is controlled by the `effort` / `reasoning.effort` / `thinking_level` parameter, not by telling the model to "think hard" or "be thorough". But depth and verbosity are separate knobs: on Claude Opus 5, lowering effort reduces thinking volume "without reliably shortening the visible response," so response length has to be prompted explicitly in Component 5.
6. **Some legacy instructions are a *delete*, not a rewrite.** When a model has absorbed a behavior into its defaults, the instruction that used to elicit it becomes a cost — self-check clauses on Opus 5 / 5.5 are the canonical case, joined by "think carefully" lines on Opus 5.5, anti-formatting blocks and "hold all findings" lines on Fable 5.1, and sampling instructions on Gemini 3.x. Softening the wording does not help; removing the section does.
7. **Leaner is shared; the specifics are not.** The three leaner-prompt vendors agree on direction, but they disagree on *which* sections to drop, and some guidance inverts between generations or vendors (Opus 4.8 needed a subagent nudge, Opus 5 needs a cap, GPT-6 needs a nudge again; earlier models overused formatting, Fable 5.1 underuses it). Read the per-model file before trimming — `claude-considerations.md` > "Pick the model before you tune" and `runtime-detection.md` > "Cross-vendor divergence" are the tables.

This does not weaken the framework — a precise, well-structured prompt is exactly what "outcome-first" prompting means. It reframes success as *sufficiency*, not completeness. The skill's tier caps (`SKILL.md` > Step 2) already encode this: the goal of the dialogue is the smallest prompt that makes the intent unambiguous.

---

## Final Prompt Template

```xml
<role>
You are a [ROLE] specializing in [DOMAIN].
</role>

<documents>
  <document index="1">
    <source>[filename]</source>
    <document_content>
      [content]
    </document_content>
  </document>
</documents>

<task>
[ACTION] — the goal is [SUCCESS CRITERIA].
</task>

<examples>
  <example>[Example 1 of desired output]</example>
  <example>[Example 2 — different scenario]</example>
  <example>[Example 3 — edge case]</example>
</examples>

<output_spec>
Format: [type + length]
Tone: [desired style, described positively]
Intended audience impact: [what effect the output should have]
Avoid: [anti-patterns, secondary]
Success: [measurable outcome]
</output_spec>

<constraints>
- [RULE]: [WHY]
- [RULE]: [WHY]
Flag any potential rule conflict before proceeding.
</constraints>
```

**Template order matters:** Documents and context go at the top, task and instructions at the bottom. This follows the guide's recommendation for optimal long-context performance.

**The template above is the XML form.** It is the Claude default and also the right default when the vendor is unresolved (`universal-baseline.md` > rule 2). For a Gemini or Grok target, the same components in the same order with Markdown headings are equivalent; `runtime-detection.md` says which applies.

**The template is a superset, not a checklist.** Include only the sections the tier and the dialogue actually produced — a quick-tier prompt with `<task>` and `<output_spec>` alone is a correct result, not an incomplete one.

---

## Adaptation Guidelines

See the [Task Tiers](#task-tiers) table above for the canonical tier-to-components mapping. The short version:

- **Quick tier (2 components):** Task + Output specification
- **Standard tier (3-4 components):** + Examples or Constraints
- **Complex tier (5+ components):** + Role, Context, all Constraints with motivation

`SKILL.md` uses these tiers to cap the dialogue length, keeping short tasks short.

For each component's formal definition, see `component-definitions.md`. For deterministic classification (what counts as `[OK]`, `[~~]`, `[--]`), see `component-rubrics.md`.

For model-specific guidance, start at `runtime-detection.md`: it maps the host to the model family and holds the cross-vendor divergence table. From there, `claude-considerations.md` routes the Anthropic branch (then `claude-opus-5-5.md` or `claude-fable-5.md`), `codex-considerations.md` covers OpenAI Codex, `gemini-considerations.md` covers Google Gemini, `grok-considerations.md` covers xAI Grok, and `universal-baseline.md` is the portable answer when the vendor cannot be resolved.

For techniques that prevent hallucinations and improve factual accuracy (investigate before answering, ground in quotes, self-check), see `grounding-techniques.md`. Note that the self-check technique is model-gated — it is omitted when the target is Claude Opus 5 / 5.5.

---

## Maintenance

The verification record (source fingerprint, models verified, date) and the step-by-step update procedure live in `maintenance.md`. Follow that procedure whenever the official guide is updated or a new model ships; it also carries the file-budget check that keeps every reference file loadable.
