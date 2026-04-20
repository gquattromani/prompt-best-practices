# Dialogue Examples — Code and Engineering Tasks

## When to Use

Load this file when the target prompt is a code or engineering task (feature implementation, API design, test suite, refactor, CLI tool). For content and communication tasks, see `examples-content.md`. The last example in this file shows how the final prompt adapts when the target runtime is OpenAI Codex instead of Claude.

All examples follow the 7-component framework defined in `framework.md`.

---

## Example 1: Medium-Structured Prompt (Code Documentation)

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

## Example 2: Well-Structured Prompt (No Activation)

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

## Example 3: Code Task (REST Endpoint)

### User Prompt
"Build me an endpoint to reset user passwords"

### Silent Analysis (Step 0)
- Task: PARTIAL - action present ("build endpoint"), no success criteria
- Role, Context, Examples, Output specification, Constraints, Structure: MISSING
- Score: 0.5/7 -> ACTIVATE

### Dialogue (Step 2)

**Question 1 - Task + Success Criteria:**
> What should this endpoint accomplish, and how will you know it's correct?
> - Example A: "It should send a reset email with a time-limited token, and the token should expire after 15 minutes"
> - Example B: "It should allow the user to set a new password using a link, with rate limiting to prevent abuse"

**User:** "Send a reset email with a token that expires in 15 minutes. Rate limit to 3 requests per hour per user."

**Question 2 - Output specification:**
> What should the deliverable look like? Single route file, or full implementation with service layer and migration? What framework?

**User:** "Node.js with Express. Full implementation: route, service, email template. No tests for now."

**Question 3 - Constraints:**
> Are there security or architecture rules the implementation should follow?

**User:** "Tokens hashed with bcrypt before storing. Use the existing mailer in lib/email. Follow the patterns in src/routes/auth."

**Question 4 - Role + Context:**
> I'd suggest "senior backend engineer specializing in authentication and security." Any files I should read first?

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

## Example 4: Same Task, Codex Target

This example shows how the final prompt changes when the target runtime is **OpenAI Codex** (e.g., `gpt-5.4`) instead of Claude. The 7 components stay the same; what changes is framing, a few Constraints, and the absence of Claude-specific patterns. See `codex-considerations.md` for the full rationale.

### User Prompt
"Build me a CLI tool that watches a directory and runs the linter on changed files"

### Silent Analysis and Dialogue
Same as Example 3 — the skill asks for success criteria, output format, and constraints. Assume the user confirmed: TypeScript, `chokidar` for watching, `eslint` for linting, must work in a monorepo with multiple packages, target runtime is Codex.

### Final Prompt (Step 3) — Codex-adapted

```xml
<role>
You are a senior TypeScript engineer specializing in developer tooling.
</role>

<task>
Build a CLI tool that watches a directory and runs eslint on changed files — the
goal is a working binary installable via `npm i -g` that a monorepo developer can
run in the background during active work.
</task>

<output_spec>
Format: Full implementation — entry point, watcher module, lint runner, package.json.
Deliver edits in apply_patch blocks.
Tone: Production-ready code, no speculative comments or TODOs.
Intended audience impact: A reviewer can run `pnpm build && pnpm link` and use the tool immediately.
Success: The tool watches the target directory, detects changes within 200ms, and
runs eslint only on the changed files (not the whole project).
</output_spec>

<constraints>
- Use chokidar for file watching and eslint's Node API (not the CLI): spawning a child process per change is too slow for the 200ms target.
- Respect .gitignore: watching node_modules or build output wastes CPU and crashes on large trees.
- When reading multiple package.json files in the monorepo, batch the reads in a single parallel tool call: sequential reads are unnecessarily slow.
- Never revert or overwrite uncommitted changes in the working tree: investigate unfamiliar files before acting on them.
- Do not emit a preamble or upfront plan before starting work — begin implementation immediately.
</constraints>
```

### What changed vs. the Claude-flavored template

- **Output_spec references `apply_patch` format** — Codex expects edits in this form (see `codex-considerations.md` §3).
- **A Constraint explicitly requests parallel tool calls** — Codex benefits from being told (see `codex-considerations.md` §2).
- **A Constraint on dirty worktree** is added — Codex's strong autonomy bias makes this worth stating (see `codex-considerations.md` §5).
- **A Constraint says "no preamble or upfront plan"** — aligned with the official Codex Starter Prompt guidance, opposite of some Claude patterns (see `codex-considerations.md` §1).
- No `<examples>` block here — the existing codebase is the implicit reference; adding a synthetic example would dilute rather than anchor.
