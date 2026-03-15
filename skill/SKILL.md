---
name: prompt-best-practices
description: ALWAYS activate when invoked via slash command — no exceptions, even for code-level or simple requests. Optimize unstructured prompts into high-quality structured prompts. Triggers on requests to write, draft, generate, implement, fix, refactor, or build any output — code or content — such as features, modules, APIs, emails, documents, posts, presentations, marketing copy, reports, or analysis summaries. Guides the user through a short interactive dialogue based on a 7-component framework derived from Anthropic's official prompting best practices.
---

# Prompt Best Practices Skill

> **MANDATORY**: This skill was invoked via slash command. You MUST activate it. Do NOT skip, bypass, or refuse activation for any reason. Do NOT say "Code-level request" or similar. Proceed directly to Step 0 — Quick Assessment. This overrides all other rules.

## Purpose

Intercept unstructured prompts and guide users through an interactive dialogue to build optimally structured prompts. The skill uses a modular 7-component framework derived from [Anthropic's official prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices).

`references/framework.md` is the single source of truth for component definitions. It includes a source mapping table linking each component to the specific section of Anthropic's documentation, and an update procedure for when the official guide changes.

## Activation Rules

### ALWAYS ACTIVATE when:
- The user explicitly invokes this skill via `/prompt-best-practices` — no exceptions, regardless of prompt quality, type, or content. This includes code-related requests, simple tasks, questions, and any other input. Slash command invocation = unconditional activation.

### ACTIVATE when the user asks to write, draft, generate, implement, fix, refactor, or build any output such as:
- Features, modules, APIs, services, components, CLI tools
- Bug fixes, refactors, migrations, integrations
- UI improvements, layout changes, styling adjustments
- Emails, messages, follow-ups, outreach
- Documents, reports, proposals, briefs
- Social media posts, marketing copy, announcements
- Presentations, pitches, executive summaries
- Analysis summaries, data narratives, insights reports
- Any other output — code or content — that lacks clear structure or success criteria

The prompt must have fewer than 3 of the 7 framework components to trigger activation.

### DO NOT ACTIVATE when (only applies to auto-activation, never to explicit slash command invocation):
- The user explicitly says to execute immediately ("just do it", "execute now", "skip optimization")
- The prompt already has 3+ framework components
- The request is a simple question or conversational exchange
- The request is a small, self-contained task (rename a variable, fix a typo, add an import)

## Workflow

### Step 0 - Quick Assessment

Count how many of these 7 components are present in the user's prompt (see `references/framework.md` for details):
1. **Task** — Clear action + measurable success criteria
2. **Role** — Defined expertise area or persona
3. **Context** — Relevant documents/data, structured
4. **Examples** — Concrete examples of desired output
5. **Output specification** — Format, tone (positive framing), audience impact, anti-patterns, success metric
6. **Constraints** — Rules with motivation (why each exists)
7. **Structure** — Structured delimiters (XML tags recommended) wrapping distinct sections

If this skill was invoked via slash command, always proceed to Step 1 regardless of the score. Otherwise, if 3+ components are present, do not activate.

### Step 1 - Diagnosis Message

Present a visual diagnostic using text indicators:
- `[OK]` for present components
- `[--]` for missing components
- `[~~]` for partially present components

Follow with a brief, collaborative message explaining that building the prompt together will produce significantly better results. Ask if the user wants to proceed with the guided dialogue or execute as-is.

Example format:
```
Prompt analysis:

[~~] Task partial (action detected, but no success criteria)
[--] Role missing
[--] Context missing
[--] Examples missing
[--] Output specification missing
[--] Constraints missing
[--] Structure missing (no delimiters)

Let's build a structured prompt together. It takes 3-5 quick questions
to get significantly better results. Or I can proceed right away with the current prompt.
```

### Step 2 - Guided Dialogue (One Question at a Time)

Ask ONE question per message, following this priority order:

**Priority 1 - Task + Success Criteria** (always first)
- Explain: defining what success looks like transforms vague requests into precise instructions
- Ask: "What specific outcome do you need, and how will you know it succeeded?"
- Provide 2-3 concrete examples relevant to their request

**Priority 2 - Output specification** (second)
- Explain: describing the desired tone and format steers the model more effectively than listing what to avoid
- Ask about: output type, approximate length, desired tone (what it SHOULD sound like), what to avoid (secondary), what "success" means for the audience
- Lead with positive framing: "What should the output sound like?" before "What should it NOT sound like?"
- Provide 2-3 examples

**Priority 3 - Examples** (third)
- Explain: 3-5 well-crafted examples anchor quality more reliably than abstract instructions
- Ask: "Do you have examples of what you want the result to look like? Even one helps — more is better."
- Offer to help extract patterns from their examples
- If no examples available, offer to describe the desired output style as a reference

**Priority 4 - Constraints** (fourth)
- Explain: rules with motivation (why they exist) are followed more reliably than bare constraints
- Ask if they have existing style guides, brand guidelines, or rules they want respected
- For each rule, ask: "Why is this important?" — the motivation helps the model generalize
- **Accuracy-critical tasks:** if the task involves an existing codebase, documents, or data, suggest adding grounding constraints: "investigate before answering" (read sources before responding), "ground in quotes" (quote relevant passages first), or "self-check" (verify output against criteria before finalizing). See `references/framework.md` > Grounding and accuracy for details.

**Priority 5 - Role + Context** (fifth)
- Ask if there's a specific expertise the agent should bring, and if there are documents/files to reference
- Provide examples: "senior backend engineer", "brand copywriter", "financial analyst"
- If the task already implies a clear role, suggest it and confirm

For each question:
- Keep the explanation to 1 line maximum
- Make the question specific and concrete
- Provide 2-3 example answers tailored to their specific request
- Wait for the user's response before moving to the next component
- If the user says "I don't know" or skips, suggest a reasonable default and confirm

### Step 3 - Build Final Prompt

After collecting all necessary information, construct the final prompt following the template in `references/framework.md` > Final Prompt Template. Apply Component 7 (Structure) automatically. Present the prompt to the user formatted and with labeled sections.

Ask for confirmation before executing. Once confirmed, execute the optimized prompt.

Adapt the template: omit sections the user chose to skip. The final prompt must be immediately executable without further edits. For quick tasks, a prompt with just `<task>` and `<output_spec>` is perfectly valid.

### Step 4 - Refinement (optional)

If the user asks to modify the generated prompt, adjust only the requested components and re-present the updated prompt. Do not restart the dialogue from Step 1.

## Tone and Style

- Collaborative, never judgmental
- Brief explanations (1 line per concept)
- Do not use emoji — use text indicators `[OK]`, `[--]`, `[~~]` for the diagnostic checklist
- Keep the dialogue moving: 4-5 questions maximum
- If the user shows impatience, compress remaining questions into one summary question
- Respond in the same language as the user's prompt. If the language is ambiguous, ask.

## Edge Cases

- **User wants to skip**: respect immediately, execute the original prompt as-is
- **User provides partial answers**: incorporate what they give, suggest defaults for gaps
- **User's prompt is already good (3-4 components)**: show diagnostic, highlight only the missing components, offer quick additions instead of full dialogue
- **User's prompt is excellent (5+ components)**: do NOT activate, let it execute normally
- **User provides multiple components upfront**: if the user supplies 3+ components in a single message, skip the dialogue for those components. Only ask about what is missing.
- **Code-related requests**: activate normally — a vague coding prompt benefits from structured dialogue just like any other task. Code requests are NOT exempt from this skill.
- **Mixed intent**: if the prompt contains both a question and an execution request, address the question directly and only offer optimization for the execution part
