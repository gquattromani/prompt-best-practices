# Prompt Structuring Framework

## When to Use

This is the single source of truth for the 7-component framework. Load it whenever you need the formal definition of a component, the task-tier mapping, the prompt-assembly template, the source-mapping table to Anthropic's guide, or the update procedure for new guide revisions.

For the operational checklist that turns "is this component present?" into a deterministic answer, see `component-rubrics.md`.

## Maintenance

| Field | Value |
|---|---|
| Source | [Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| Last verified | 2026-06-16 |
| Verified against | Claude Opus 4.8 (default, `claude-opus-4-8`), Claude Sonnet 4.6. Newest siblings: Claude Fable 5 / Claude Mythos 5. |
| Source fingerprint | Page restructured into three parts (model-specific guidance / techniques for all current models / migration). Model-specific guidance now lives on dedicated pages (`prompting-claude-opus-4-8`, `prompting-claude-fable-5`), no longer in-page anchors. General-principles anchors stable: `be-clear-and-direct`, `give-claude-a-role`, `long-context-prompting`, `use-examples-effectively`, `control-the-format-of-responses`, `add-context-to-improve-performance`, `structure-prompts-with-xml-tags`. New "Communication style and verbosity" subsection under Output and formatting (length calibration moved to the per-model pages). Recompute on update. |

When the source page is updated, follow the [update procedure](#update-procedure) at the bottom of this file. The procedure includes a fingerprint diff so a contributor can tell at a glance whether the source page shifted meaningfully since the last verification.

---

## Overview

This framework defines 7 components for building effective prompts. Each component is derived from a specific section of [Anthropic's official prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices).

The framework is **modular, not prescriptive**. The best approach depends on the type of task — not every prompt needs all 7 components. Use what adds value and skip what does not.

### Source mapping

Each component links to the section of the official guide it derives from.

| # | Component | Source section | Key quote |
|---|---|---|---|
| 1 | Task | [General principles > Be clear and direct](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#be-clear-and-direct) | "Being specific about your desired output can help enhance results." |
| 2 | Role | [General principles > Give Claude a role](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#give-claude-a-role) | "Setting a role in the system prompt focuses Claude's behavior and tone." |
| 3 | Context | [General principles > Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting) | "Put longform data at the top. Queries at the end can improve response quality by up to 30%." |
| 4 | Examples | [General principles > Use examples effectively](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#use-examples-effectively) | "A few well-crafted examples can dramatically improve accuracy and consistency." |
| 5 | Output specification | [Output and formatting > Control the format of responses](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#control-the-format-of-responses) | "Tell Claude what to do instead of what not to do." |
| 6 | Constraints | [General principles > Add context to improve performance](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#add-context-to-improve-performance) | "Explaining to Claude why such behavior is important can help Claude better understand your goals." |
| 7 | Structure | [General principles > Structure prompts with XML tags](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#structure-prompts-with-xml-tags) | "XML tags help Claude parse complex prompts unambiguously." |

### Task Tiers

The framework is **modular, not prescriptive**. "Modular" is operationalized through three task tiers. Each tier sets the *required* component count and the *dialogue cap* used by `SKILL.md` > Step 2. A prompt meeting or exceeding its tier threshold does not need auto-activation; slash-command invocation still assesses but fast-tracks quickly.

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

### Additional source mapping

Additionally, `references/grounding-techniques.md` and `references/claude-considerations.md` derive from:
- [Thinking and reasoning > Overthinking and excessive thoroughness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#overthinking-and-excessive-thoroughness)
- [Agentic systems > Overeagerness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#overeagerness)
- [Tool use > Tool usage](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#tool-usage)
- [Agentic systems > Minimizing hallucinations in agentic coding](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#minimizing-hallucinations-in-agentic-coding)
- [General principles > Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting) (grounding in quotes)
- [Thinking and reasoning > Leverage thinking](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#leverage-thinking--interleaved-thinking-capabilities) (self-check)

---

## Component 1: Task

> Source: [Be clear and direct](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#be-clear-and-direct)
> "Claude responds well to clear, explicit instructions. Being specific about your desired output can help enhance results. If you want 'above and beyond' behavior, explicitly request it rather than relying on the model to infer this from vague prompts."

**What it is:** A specific action with measurable or observable success criteria.

**Format:**
```xml
<task>
Write a follow-up email to a client — the goal is getting them to book a call within 48 hours.
</task>
```

**What to check for:**
- Is there a clear, specific action verb? (write, build, analyze, design, draft)
- Is there a measurable or observable success criteria?
- Does the task describe the "what" AND the "why"?

**Present if:** The prompt clearly states both what to do and what success looks like.
**Missing if:** The prompt only says what to do without defining success, or is vague about the actual task.

**Good examples:**
- "Write a follow-up email to a client — the goal is getting them to book a call within 48 hours."
- "Create landing page copy. Success looks like: visitors understand the value proposition in under 10 seconds."
- "Refactor the authentication module to pass all security audit requirements."

**Bad examples (missing success criteria):**
- "Write me an email" (no success criteria, no specificity)
- "Create a landing page" (what outcome? for whom?)
- "Fix the auth system" (what does "fixed" look like?)

---

## Component 2: Role

> Source: [Give Claude a role](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#give-claude-a-role)
> "Setting a role in the system prompt focuses Claude's behavior and tone for your use case. Even a single sentence makes a difference."

**What it is:** A defined persona or expertise area that focuses the model's behavior.

**Format:**
```xml
<role>
You are a senior backend engineer specializing in API design and security.
</role>
```

**What to check for:**
- Is there a defined persona or expertise area?
- Does the role match the task requirements?

**Present if:** The prompt defines a role, persona, or area of expertise.
**Missing if:** No role is set — the model uses its default generalist behavior.

**When to use:** Most beneficial for specialized tasks (legal review, medical summaries, brand copywriting, security audits). Less critical for straightforward tasks where the model's default behavior is sufficient.

---

## Component 3: Context

> Source: [Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting)
> "Put your long documents and inputs near the top of your prompt, above your query, instructions, and examples. This can significantly improve performance across all models."
> "Structure document content and metadata with XML tags."
> "For long document tasks, ask Claude to quote relevant parts of the documents first before carrying out its task."

**What it is:** Relevant documents, data, or background information — structured with XML tags and placed at the top of the prompt.

**Format:**
```xml
<documents>
  <document index="1">
    <source>style-guide.md</source>
    <document_content>
      {{STYLE_GUIDE}}
    </document_content>
  </document>
</documents>
```

**What to check for:**
- Does the prompt reference external files or inline context?
- Is the context separated from the instructions using XML tags?
- Are documents placed before the task/query (top of prompt)?

**Present if:** The prompt includes relevant background information, structured with XML tags.
**Missing if:** All context is inline without structure, or there is no context at all.

**Types of context:**
- Style guides (brand voice, tone, formatting rules)
- Audience profiles (who will read/use the output)
- Domain expertise (technical constraints, industry standards)
- Project-specific rules (coding standards, design system tokens)
- Existing codebase or documentation to reference

---

## Component 4: Examples

> Source: [Use examples effectively](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#use-examples-effectively)
> "Examples are one of the most reliable ways to steer Claude's output format, tone, and structure. A few well-crafted examples (known as few-shot or multishot prompting) can dramatically improve accuracy and consistency."
> "Include 3-5 examples for best results."

**What it is:** Concrete examples of desired output, wrapped in `<example>` tags.

**Format:**
```xml
<examples>
  <example>
    <input>Customer complaint about shipping delay</input>
    <output>Response acknowledging the issue, providing tracking update, offering discount code</output>
  </example>
  <example>
    <input>Customer asking for refund on damaged item</input>
    <output>Empathetic response, immediate refund confirmation, return label attached</output>
  </example>
</examples>
```

**What to check for:**
- Are there concrete examples of the desired output?
- Are examples wrapped in `<example>` tags?
- Are there at least 3 examples covering different scenarios?

**Present if:** There is at least one structured example of expected output.
**Missing if:** No examples are provided.

**Best practices (from the guide):**
- Provide **3-5 examples** for best results.
- Make them **relevant** — mirror your actual use case closely.
- Make them **diverse** — cover edge cases so the model doesn't pick up unintended patterns.
- Wrap in `<example>` tags so the model distinguishes them from instructions.

---

## Component 5: Output specification

> Source: [Control the format of responses](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#control-the-format-of-responses)
> "Tell Claude what to do instead of what not to do."
> "The formatting style used in your prompt may influence Claude's response style."

**What it is:** A description of the desired output — format, length, tone, and success metric. Leads with positive framing (what the output SHOULD be) before anti-patterns.

**Format:**
```xml
<output_spec>
Format: Professional email, 150-200 words
Tone: Like a trusted advisor — professional but warm, direct but respectful
Intended audience impact: The client should feel valued and compelled to respond
Avoid: Generic template tone, pressure tactics, formulaic closings
Success: The client replies within 48h with a confirmed meeting time
</output_spec>
```

**What to check for:**
- Output type and approximate length specified?
- Desired tone described in positive terms (what it SHOULD sound like)?
- Impact on the audience defined?
- Anti-patterns listed as secondary guidance?
- Clear success metric?

**Present if:** At least 3 of the 5 sub-fields are specified.
**Partially present if:** 1-2 sub-fields are specified.
**Missing if:** None are specified.

**Framing principle:** Lead with positive descriptions. "Write in flowing prose with complete paragraphs" is more effective than "Don't use bullet points." Anti-patterns are useful but secondary.

**On the coexistence of `Tone` and `Avoid`:** the official guide's "tell what to do, not what not to do" is a *framing priority*, not a prohibition on anti-patterns. Positive descriptions generalize well to the whole output space; anti-patterns are a targeted disambiguator for phrasings the model is known to fall into (e.g., "hope this helps" closings, collection-notice tone, formulaic hedges). The rule is:

- `Tone` is **required** when output_spec is used — it sets the primary direction.
- `Avoid` is **optional and secondary** — include only when there is a specific anti-pattern worth naming. Never use it as a substitute for positive framing.

If `Avoid` ends up longer than `Tone`, the spec is upside-down; rewrite `Tone` until it covers the same ground positively.

---

## Component 6: Constraints

> Source: [Add context to improve performance](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#add-context-to-improve-performance)
> "Providing context or motivation behind your instructions, such as explaining to Claude why such behavior is important, can help Claude better understand your goals and deliver more targeted responses."

**What it is:** Explicit rules with motivation (why each rule exists). Models follow rules more reliably when they understand the reason.

**Format:**
```xml
<constraints>
- Never use ellipses: the output will be read by a text-to-speech engine that cannot pronounce them.
- Keep paragraphs under 4 lines: the audience reads on mobile devices with small screens.
- Use only metric units: this document targets a European audience.

Flag any potential rule conflict before proceeding.
</constraints>
```

**What to check for:**
- Are rules stated as clear constraints?
- Does each rule include a brief motivation (why it exists)?
- Is there a conflict-flagging clause?

**Present if:** Rules are stated with motivation.
**Missing if:** No rules, or rules without motivation.

**Key insight from the guide:** Instead of "NEVER use ellipses", write "Never use ellipses: the output will be read by a text-to-speech engine that cannot pronounce them." The model generalizes better from the explanation than from the bare rule.

---

## Component 7: Structure

> Source: [Structure prompts with XML tags](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#structure-prompts-with-xml-tags)
> "XML tags help Claude parse complex prompts unambiguously, especially when your prompt mixes instructions, context, examples, and variable inputs. Wrapping each type of content in its own tag reduces misinterpretation."

**What it is:** The use of structured delimiters to wrap distinct sections of the prompt, giving each part a clear semantic label. Structure is *format-agnostic* — what matters is unambiguous separation, not which syntax provides it.

**Format options (all valid):**

| Format | When to prefer | Example |
|---|---|---|
| **XML tags** | Claude (default, strongest-tested) | `<task>...</task>` |
| **Markdown headings** | Codex, models that train heavily on Markdown, human-edited prompts | `## Task\n...` |
| **JSON** | Programmatic prompt assembly, strict schemas | `{"task": "...", "output_spec": {...}}` |
| **Fenced blocks with labels** | Inline contexts, chat UIs that strip XML | ` ```task ... ``` ` |

Pick one and use it consistently across the prompt. Mixing formats (XML for some sections, Markdown for others) defeats the purpose.

**What to check for:**
- Are distinct sections of the prompt wrapped in descriptive delimiters?
- Are delimiter names consistent and descriptive?
- Is content with a natural hierarchy nested properly?
- Is the chosen format appropriate for the target runtime?

**Present if:** The prompt uses consistent structured delimiters to separate at least 2 distinct sections.
**Missing if:** The prompt is a flat block of text with no structural markers.

**Runtime-specific defaults:**
- Claude → XML tags (see `claude-considerations.md`). Parses unambiguously; strongest-tested.
- Codex → XML or Markdown headings, both work well (see `codex-considerations.md`).
- Unknown / mixed fleet → XML is the safest single choice.

**Best practices (from the guide):**
- Use consistent, descriptive names across your prompts.
- Nest when content has a natural hierarchy (documents inside `<documents>`, each inside `<document index="n">`).

**Why it matters:** Structure is the only component that doesn't add content — it improves how the model parses the other 6 components.

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

---

## Adaptation Guidelines

See the [Task Tiers](#task-tiers) table above for the canonical tier-to-components mapping. The short version:

- **Quick tier (2 components):** Task + Output specification
- **Standard tier (3-4 components):** + Examples or Constraints
- **Complex tier (5+ components):** + Role, Context, all Constraints with motivation

`SKILL.md` uses these tiers to cap the dialogue length, keeping short tasks short.

For deterministic component classification (what counts as `[OK]`, `[~~]`, `[--]`), see `component-rubrics.md`.

For model-specific guidance, see `claude-considerations.md` (Anthropic Claude) and `codex-considerations.md` (OpenAI Codex).

For techniques that prevent hallucinations and improve factual accuracy (investigate before answering, ground in quotes, self-check), see `grounding-techniques.md`.

---

## Update procedure

Follow this procedure when the official guide is updated.

1. **Fetch the latest version** of the source page:
   `https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices`

2. **Recompute the source fingerprint.** Compare against the `Source fingerprint` row in the maintenance table. The fingerprint is a short checklist of the named sections and key quotes the framework depends on. If any fingerprint item is missing or materially changed, the framework needs patching.

3. **Check each component against its source section.** Use the [source mapping table](#source-mapping) to verify each component is still aligned. For each row:
   - Does the source section still exist at the linked anchor?
   - Has the key quote changed or been removed?
   - Are there new recommendations that should be reflected?

4. **Check for new sections.** Scan the guide for sections not covered by any existing component. If a new principle is introduced, evaluate whether it warrants a new component or an update to an existing one.

5. **Check the Claude-specific considerations.** Model-specific guidance now lives on dedicated `prompting-claude-<model>` pages (e.g., `prompting-claude-opus-4-8`, `prompting-claude-fable-5`), not in-page anchors. When a new flagship ships, update `references/claude-considerations.md` to track it (rename the section, refresh the numbered points, update the per-model page links).

6. **Re-run the activation fixtures.** `tests/activation-fixtures.md` lists reference prompts with expected activation outcomes. Any edit to `SKILL.md` or this file must leave all fixtures producing the expected outcome.

7. **Update the maintenance table** at the top of this file with the new verification date, model generation, and recomputed fingerprint.

8. **Update SKILL.md** if any component was added, removed, or renamed — the component list in Step 0, the tier mapping, and the dialogue priorities in Step 2 must match this file.

9. **Update `component-rubrics.md`** if the definition of any component changed — the operational checklist has to reflect the new definition.

10. **Update `examples-content.md` and `examples-code.md`** if the scoring denominator changed (e.g., `/7` → `/8`) or if example final prompts need to reflect new template structure.

11. **Update the version** in `plugin.json` (single source of truth) and add a new entry in `CHANGELOG.md`.

### Important note

This framework is a practical tool, not a rigid prescription. The primary source for the principles behind it is [Anthropic's prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). The best prompting approach always depends on the specific task — use the components that add value and skip those that do not. A well-written 2-component prompt will always outperform a poorly thought-out 7-component one.
