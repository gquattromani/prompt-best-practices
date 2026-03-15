# Prompt Best Practices - Dialogue Examples

This file contains complete example dialogues demonstrating how the skill analyzes prompts and guides users through the optimization process. All examples follow the 7-component framework defined in `references/framework.md`.

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

## Example 2: Medium-Structured Prompt (Code Documentation)

### User Prompt
"Generate the API documentation for the authentication module. It should be in OpenAPI 3.0 format, cover all endpoints, and include request/response examples."

### Silent Analysis (Step 0)
- Task: PRESENT - clear task with implicit success criteria
- Role: MISSING
- Context: MISSING
- Examples: MISSING
- Output specification: PARTIAL - format specified (OpenAPI 3.0), length implied, but no tone or anti-patterns
- Constraints: MISSING
- Structure: MISSING
- Score: 1.5/7 -> ACTIVATE (but lighter touch needed)

### Diagnosis (Step 1)
```
Prompt analysis:

[OK] Task present (API documentation for authentication module)
[--] Role missing
[--] Context missing
[--] Examples missing
[~~] Output specification partial (OpenAPI 3.0 format, request/response examples)
[--] Constraints missing
[--] Structure missing

Your prompt is already partially structured. It takes 2-3 questions to complete it.
Do you want to proceed or execute right away?
```

### Dialogue (Step 2)
Since the prompt is already partially structured, the dialogue is shorter:

**Question 1 - Output specification + Success criteria:**
> What makes this documentation "complete" for you, and what tone should it have?
> - Example: "It should allow a junior dev to integrate auth without asking for help"
> - Example: "Clear, concise — not a wall of text with no structure"

**User:** "A junior dev should be able to integrate auth without asking questions. Keep it concise, no filler."

**Question 2 - Examples + Context:**
> Do you have an example of API documentation you consider well-made? Are there project-specific conventions (naming, auth schemes, error format)?

**User:** "We use Bearer tokens, error responses follow RFC 7807. Look at how Stripe's API docs are structured — that's the standard."

### Final Prompt (Step 3)
```xml
<role>
You are a senior technical writer specializing in API documentation.
</role>

<documents>
  <document index="1">
    <source>authentication module source code</source>
    <document_content>
      {{AUTH_MODULE_CODE}}
    </document_content>
  </document>
</documents>

<task>
Generate complete API documentation for the authentication module in OpenAPI 3.0
format — the goal is that a junior developer can integrate auth without asking
for help.
</task>

<examples>
  <example>
    Follow Stripe's API documentation structure: endpoint summary, authentication
    method, request parameters table, response schema, and a concrete curl example
    for each endpoint.
  </example>
</examples>

<output_spec>
Format: OpenAPI 3.0 YAML, covering all endpoints with request/response examples
Tone: Concise and precise, no filler text
Intended audience impact: A junior developer reads it and integrates successfully on first attempt
Avoid: Verbose descriptions, placeholder values, incomplete schemas
Success: Every endpoint is documented with auth method, parameters, response schema, and a working example
</output_spec>

<constraints>
- Use Bearer token authentication in all examples: this is the project's auth scheme.
- Error responses must follow RFC 7807 (Problem Details): this is the project standard.
- Read the authentication module source code before writing: ground the documentation in actual implementation, not assumptions.
</constraints>
```

---

## Example 3: Well-Structured Prompt (No Activation)

### User Prompt
```xml
<role>
You are a senior Swift engineer specializing in test architecture.
</role>

<task>
Create a comprehensive test suite for the PaymentService class — the goal is
100% branch coverage so the team can refactor with confidence.
</task>

<examples>
  <example>Follow the testing patterns in tests/OrderServiceTest.swift.</example>
</examples>

<output_spec>
Format: Swift Testing test file covering all public methods + edge cases
Tone: Clear test names that read as specifications
Avoid: Trivial tests that only check happy paths
Success: 100% branch coverage on PaymentService
</output_spec>

<constraints>
- Read TESTING_GUIDELINES.md before starting: it contains project-specific conventions that override general best practices.
- Flag any potential conflict with testing conventions before proceeding.
</constraints>
```

### Silent Analysis (Step 0)
- Task: PRESENT (create test suite + measurable success criteria)
- Role: PRESENT (senior Swift engineer)
- Context: PRESENT (TESTING_GUIDELINES.md, OrderServiceTest.swift)
- Examples: PRESENT (OrderServiceTest.swift as reference)
- Output specification: PRESENT (format, tone, anti-pattern, success metric)
- Constraints: PRESENT (read guidelines + conflict-flagging with motivation)
- Structure: PRESENT (XML tags throughout)
- Score: 7/7 -> DO NOT ACTIVATE

The skill does not activate. The prompt is executed as-is.

---

## Example 4: Social Media Post

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

## Example 5: Data Analysis

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

## Example 6: Code Task (REST Endpoint)

### User Prompt
"Build me an endpoint to reset user passwords"

### Silent Analysis (Step 0)
- Task: PARTIAL - action present ("build endpoint"), no success criteria
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

[~~] Task partial (build password reset endpoint — but no success criteria)
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
> What should this endpoint accomplish, and how will you know it's correct?
> - Example A: "It should send a reset email with a time-limited token, and the token should expire after 15 minutes"
> - Example B: "It should allow the user to set a new password using a link, with rate limiting to prevent abuse"
> - Example C: "It should integrate with our existing auth flow and pass all security audit checks"

**User:** "Send a reset email with a token that expires in 15 minutes. Rate limit to 3 requests per hour per user."

**Question 2 - Output specification:**
> Describing the expected output helps scope the work precisely.
>
> What should the deliverable look like?
> - A single controller/route file?
> - Full implementation with service layer, tests, and migration?
> - What framework and language?

**User:** "Node.js with Express. Full implementation: route, service, email template. No tests for now."

**Question 3 - Constraints:**
> Rules with motivation are followed more reliably than bare requirements.
>
> Are there security or architecture rules the implementation should follow?
> - Example: "Tokens must be hashed before storage"
> - Example: "Use the existing mailer service, don't add a new dependency"

**User:** "Tokens hashed with bcrypt before storing. Use the existing mailer in lib/email. Follow the patterns in src/routes/auth."

**Question 4 - Role + Context:**
> Given what you've described, I'd suggest "senior backend engineer specializing in authentication and security." Does that work? Any files I should read first?

**User:** "Yes, that works. Read src/routes/auth/login.ts for the pattern."

### Final Prompt (Step 3)
```xml
<role>
You are a senior backend engineer specializing in authentication and security.
</role>

<documents>
  <document index="1">
    <source>src/routes/auth/login.ts</source>
    <document_content>
      {{LOGIN_ROUTE_CODE}}
    </document_content>
  </document>
</documents>

<task>
Build a password reset endpoint for an Express application — the goal is a
working implementation that sends a reset email with a time-limited token
(15-minute expiry) and enforces rate limiting (3 requests per hour per user).
</task>

<output_spec>
Format: Full implementation — route handler, service layer, email template
Tone: Production-ready code following existing project patterns
Intended audience impact: A reviewer can approve and merge without structural changes
Avoid: Test files, new dependencies, over-abstraction beyond what the existing auth routes do
Success: The endpoint sends a reset email, the token expires after 15 minutes, and abuse is rate-limited
</output_spec>

<constraints>
- Hash tokens with bcrypt before storing: plain-text tokens in the database are a security vulnerability.
- Use the existing mailer in lib/email: avoid adding dependencies for functionality that already exists.
- Follow the patterns in src/routes/auth/login.ts: consistency with the existing codebase reduces review friction.
- Read src/routes/auth/login.ts before writing any code: ground the implementation in actual project patterns, not assumptions.
</constraints>
```

---

## Anti-Pattern: Over-Triggering

### User Prompt (DO NOT activate)
"What is the Observer pattern in Swift?"

**Analysis:** This is a simple question, not an execution request. No imperative verb targeting output generation. Score is irrelevant -> DO NOT ACTIVATE.

### User Prompt (DO NOT activate)
"Rename the variable userList to users in the file UserViewModel.swift"

**Analysis:** This is a precise code-level task within an active codebase. The codebase provides all necessary context. -> DO NOT ACTIVATE.

### User Prompt (DO NOT activate)
"Execute immediately without optimizing: write a comment for this function"

**Analysis:** The user explicitly asked to execute without optimization. -> DO NOT ACTIVATE.
