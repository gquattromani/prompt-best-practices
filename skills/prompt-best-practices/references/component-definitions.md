# Component Definitions (1-7)

## When to Use

Load this file when you need the formal definition of a prompt component: what it is, its canonical format, what to check for, and when it is present or missing. This is the authoritative source for *what each component is*.

Two sibling files complete the picture and are often needed together:

- `framework.md` — the task tiers, the source-mapping table, the frontier-model calibration, and the final prompt template. Load it first if you have not already.
- `component-rubrics.md` — the operational checklist for *how to score* a component as `[OK]` / `[~~]` / `[--]`.

Every component below derives from a section of [Anthropic's prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices); the source link and key quote open each section.

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
> "Examples are one of the most reliable ways to steer Claude's output format, tone, and structure. A few well-crafted examples (known as few-shot or multishot prompting) improve accuracy and consistency."
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

**Frontier-model note:** on Claude Opus 5, Claude Fable 5 / Mythos 5, and OpenAI GPT-5.6, few-shot examples are the *first* component to reconsider. These models infer format and intent from a clear task, so a redundant example adds tokens and noise rather than accuracy. Include an example only when it demonstrably steers something words cannot (a subtle format, a specific edge case). One exception worth keeping: when tuning *communication style* on Opus 5, a positive example of the voice you want beats a list of things to avoid (`claude-opus-5.md` § 6). See `framework.md` > "Calibrating for frontier models".

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

**Frontier-model note:** this component gained weight on Claude Opus 5. Its default responses run longer than prior models', and `effort` controls thinking volume rather than visible length — so `Format` (length) and `Tone` (concision) are the only levers, and they matter even on quick-tier prompts. See `claude-opus-5.md` § 1.

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

**Frontier-model note:** on Claude Opus 5 this component carries more weight, but its content shifts. Scope and delegation limits earn their place ("change only the file named below", "do the work yourself unless the task splits into independent workstreams"); verification and self-check clauses must be *removed* rather than softened. See `claude-opus-5.md` § 2-§ 5 and `grounding-techniques.md` technique 3.

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
