# Prompt Structuring Framework

## Maintenance

| Field | Value |
|---|---|
| Source | [Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| Last verified | 2026-04-13 |
| Verified against | Claude 4.6 (Opus, Sonnet) |

When the source page is updated, follow the [update procedure](#update-procedure) at the bottom of this file.

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

Additionally, the [Grounding and accuracy](#grounding-and-accuracy) section and `references/claude-considerations.md` derive from:
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

**What it is:** The use of structured delimiters to wrap distinct sections of the prompt, giving each part a clear semantic label. XML tags are the recommended format (and the most tested), but Markdown headings, triple backticks, or other consistent markers also work.

**What to check for:**
- Are distinct sections of the prompt wrapped in descriptive delimiters?
- Are delimiter names consistent and descriptive?
- Is content with a natural hierarchy nested properly?

**Present if:** The prompt uses structured delimiters to separate at least 2 distinct sections.
**Missing if:** The prompt is a flat block of text with no structural markers.

**Best practices (from the guide):**
- Use consistent, descriptive names across your prompts.
- Nest when content has a natural hierarchy (documents inside `<documents>`, each inside `<document index="n">`).
- **Recommended format:** XML tags provide the strongest semantic separation and are unambiguous for most models. Use them by default unless the target model or platform has a different recommendation.

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

Not every prompt needs all 7 components. The framework is modular:

- **Quick tasks (2 components):** Task + Output specification
- **Standard tasks (3-4 components):** Task + Output specification + Examples + Constraints
- **Complex tasks (5-7 components):** All components as needed

For Claude-specific guidance (model behaviors, known tendencies, version-specific tips), see `references/claude-considerations.md`.

### Grounding and accuracy

Three techniques from the official guide help prevent hallucinations and improve factual accuracy. Consider adding these as constraints (Component 6) when accuracy is critical.

**1. Investigate before answering** — from [Minimizing hallucinations in agentic coding](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#minimizing-hallucinations-in-agentic-coding):

> "Claude's latest models are less prone to hallucinations and give more accurate, grounded, intelligent answers based on the code."

When the prompt involves an existing codebase, documents, or data, add this constraint:

```xml
<constraints>
- Never speculate about content you have not read. If a specific file or source is referenced,
  read it before answering. Investigate first, then respond — give grounded, hallucination-free answers.
</constraints>
```

**2. Ground responses in quotes** — from [Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting):

> "For long document tasks, ask Claude to quote relevant parts of the documents first before carrying out its task. This helps Claude cut through the noise."

When the prompt involves long documents, add this instruction:

```
Before answering, extract and quote the relevant passages from the provided documents.
Then base your response on those quotes.
```

**3. Self-check before finalizing** — from [Leverage thinking & interleaved thinking capabilities](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#leverage-thinking--interleaved-thinking-capabilities):

> "Ask Claude to self-check. Append something like 'Before you finish, verify your answer against [test criteria].' This catches errors reliably, especially for coding and math."

When the prompt involves precise or verifiable outputs, add:

```
Before finalizing, verify your output against [specific criteria].
```

---

## Update procedure

Follow this procedure when the official guide is updated.

1. **Fetch the latest version** of the source page:
   `https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices`

2. **Check each component against its source section.** Use the [source mapping table](#source-mapping) to verify each component is still aligned. For each row:
   - Does the source section still exist at the linked anchor?
   - Has the key quote changed or been removed?
   - Are there new recommendations that should be reflected?

3. **Check for new sections.** Scan the guide for sections not covered by any existing component. If a new principle is introduced, evaluate whether it warrants a new component or an update to an existing one.

4. **Check the Claude-specific considerations.** If the guide mentions a new model generation (e.g., Claude 5.x), update `references/claude-considerations.md` with the new guidance.

5. **Update the maintenance table** at the top of this file with the new verification date and model generation.

6. **Update SKILL.md** if any component was added, removed, or renamed — the component list in Step 0 and the dialogue priorities in Step 2 must match framework.md.

7. **Update examples.md** if the scoring denominator changed (e.g., `/7` → `/8`) or if example final prompts need to reflect new template structure.

8. **Update the version** in `plugin.json` (single source of truth) and add a new entry in `CHANGELOG.md`.

### Important note

This framework is a practical tool, not a rigid prescription. The primary source for the principles behind it is [Anthropic's prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). The best prompting approach always depends on the specific task — use the components that add value and skip those that do not. A well-written 2-component prompt will always outperform a poorly thought-out 7-component one.
