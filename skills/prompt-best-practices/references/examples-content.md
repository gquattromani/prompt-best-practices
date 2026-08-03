# Dialogue Examples — Content and Communication Tasks

## When to Use

Load this file when the target prompt is a communication, writing, or analysis task (email, social post, report, analysis summary). For code and engineering tasks, see `examples-code.md`. For anti-patterns showing when the skill should *not* activate, see the section at the bottom of this file.

All examples follow the 7-component framework: `framework.md` for tiers and the assembly template, `component-definitions.md` for the component definitions.

---

## Example 1: Vague Prompt (Email)

### User Prompt
"Write me a follow-up email for a client who hasn't responded"

### Silent Analysis (Step 0)
- Task: PARTIAL - action present ("write follow-up email"), no success criteria
- Role: MISSING
- Context: MISSING
- Examples: MISSING
- Output specification: MISSING
- Constraints: MISSING
- Structure: MISSING
- Score: 0.5/7 -> ACTIVATE

### Diagnosis (Step 1)
```
Prompt analysis:

[~~] Task partial (write follow-up email — but no success criteria)
[--] Role missing
[--] Context missing
[--] Examples missing
[--] Output specification missing
[--] Constraints missing
[--] Structure missing

Let's build a structured prompt together. It takes 3-4 quick questions
to get significantly better results. Or I can proceed right away.
```

### Dialogue (Step 2)

**Question 1 - Task + Success Criteria:**
> Defining success transforms a vague request into a precise instruction.
>
> What should happen after the client reads this email?
> - Example A: "They should book a call within 48h"
> - Example B: "They should reply confirming interest in the project"
> - Example C: "They should click the link and fill out the onboarding form"

**User:** "They should reply to confirm they want to proceed with the quote"

**Question 2 - Output specification:**
> Describing the desired tone steers the model more effectively than listing what to avoid.
>
> What should this email sound like?
> - Tone: formal, semi-formal, or direct?
> - Length: 3 lines or 3 paragraphs?
> - What tone should it NOT have? (e.g. "not an aggressive reminder")

**User:** "Semi-formal, short, 5 lines max. It shouldn't feel like a demand or pressure"

**Question 3 - Examples:**
> Concrete examples anchor quality expectations. Even one reference helps significantly.
>
> Do you have an example of an email you like style-wise, or can you describe the tone you want?
> - Example A: "Like a senior consultant would write — professional but human"
> - Example B: "Like a message between colleagues who know each other well"
> - Example C: "Like a gentle notification, similar to an app reminder"

**User:** "Like a senior consultant, professional but human. No formulaic phrases like 'hoping to hear from you soon'"

**Question 4 - Constraints + Role:**
> Do you have rules about corporate communication tone? And what expertise should the agent bring?
> If not, no problem — I'll use the guidance you've already provided.

**User:** "No rules file. Just make it sound like a senior consultant."

### Final Prompt (Step 3)
```xml
<role>
You are a senior business consultant with a warm, professional communication style.
</role>

<task>
Write a follow-up email to a client who hasn't responded to a quote — the goal is
getting them to reply confirming they want to proceed.
</task>

<examples>
  <example>
    A senior consultant's follow-up: professional but human, direct but not pushy.
    Acknowledges the client's time, references the specific quote, and offers
    a clear low-friction next step.
  </example>
</examples>

<output_spec>
Format: Semi-formal email, maximum 5 lines
Tone: Professional but human, like a trusted advisor checking in
Intended audience impact: Should feel like a gentle, professional nudge, not a demand
Avoid: Collection notice tone, pushy salesperson, generic templates, formulaic closings
Success: The client replies to confirm they want to proceed
</output_spec>

<constraints>
- Always reference the specific quote or project by name: personalization increases response rates.
- Always include a clear, low-friction next step: reduces friction for the client to act.
- Never exceed 5 lines: brevity respects the client's time and increases read-through rate.
</constraints>
```

---

## Example 2: Social Media Post

### User Prompt
"Write me a LinkedIn post to announce the launch of our new product"

### Silent Analysis (Step 0)
- Task: PARTIAL - action present, no success criteria
- Everything else: MISSING
- Score: 0.5/7 -> ACTIVATE

### Abbreviated Dialogue Flow
1. **Task + Success Criteria:** "What should the reader do after seeing the post? Comment? Click a link? Share?"
2. **Output specification:** "What tone do you want? What should the output sound like? Length? What to avoid? (e.g. generic corporate announcement)"
3. **Examples:** "Do you have a LinkedIn post you like style-wise? Or a profile you admire for their communication?"
4. **Constraints:** "Is there a brand voice guide or corporate communication rules? Why are those rules important?"

---

## Example 3: Data Analysis

### User Prompt
"Analyze these sales data and give me useful insights"

### Silent Analysis (Step 0)
- Task: PARTIAL - very vague ("useful insights" is not a success criteria)
- Everything else: MISSING
- Score: 0.5/7 -> ACTIVATE

### Key Questions
1. **Task + Success Criteria:** "What decision do you need to make with these insights? What changes if the analysis goes well?"
   - E.g.: "Decide whether to invest more in the B2B channel"
   - E.g.: "Understand why Q3 had a decline"
   - E.g.: "Prepare a report for Friday's board meeting"
2. **Output specification:** "What format do you want? Dashboard, bullet points, narrative report? How long? What tone — executive summary or deep-dive technical?"
3. **Examples:** "Do you have an example of an analysis you found useful in the past?"

---

## Anti-Patterns: When NOT to Activate

### Anti-Pattern 1 — Simple question
User prompt: *"What is the Observer pattern in Swift?"*

**Analysis:** This is a simple question, not an execution request. No imperative verb targeting output generation. Score is irrelevant -> DO NOT ACTIVATE.

### Anti-Pattern 2 — Precise code-level task
User prompt: *"Rename the variable userList to users in the file UserViewModel.swift"*

**Analysis:** This is a precise code-level task within an active codebase. The codebase provides all necessary context. -> DO NOT ACTIVATE.

### Anti-Pattern 3 — Explicit skip
User prompt: *"Execute immediately without optimizing: write a comment for this function"*

**Analysis:** The user explicitly asked to execute without optimization. -> DO NOT ACTIVATE.
