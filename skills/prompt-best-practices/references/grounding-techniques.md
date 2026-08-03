# Grounding and Accuracy Techniques

## When to Use

Load this file when the prompt involves accuracy-critical work: reading an existing codebase, quoting from documents, verifying outputs against constraints, or any task where hallucinations would cause real harm. The three techniques below can be added as Constraints (Component 6) or woven into the task instruction.

**Techniques 1 and 2 are model-agnostic. Technique 3 is model-gated** — it must be omitted when the target is Claude Opus 5. Read technique 3 before adding a verification clause.

## Techniques

Three techniques from the official guide help prevent hallucinations and improve factual accuracy.

### 1. Investigate before answering

From [Minimizing hallucinations in agentic coding](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#minimizing-hallucinations-in-agentic-coding):

> "Claude's latest models are less prone to hallucinations and give more accurate, grounded, intelligent answers based on the code."

When the prompt involves an existing codebase, documents, or data, add this constraint:

```xml
<constraints>
- Never speculate about content you have not read. If a specific file or source is referenced,
  read it before answering. Investigate first, then respond — give grounded, hallucination-free answers.
</constraints>
```

### 2. Ground responses in quotes

From [Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting):

> "For long document tasks, ask Claude to quote relevant parts of the documents first before carrying out its task. This helps Claude cut through the noise."

When the prompt involves long documents, add this instruction:

```
Before answering, extract and quote the relevant passages from the provided documents.
Then base your response on those quotes.
```

### 3. Self-check before finalizing — model-gated

From [Leverage thinking & interleaved thinking capabilities](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#leverage-thinking--interleaved-thinking-capabilities):

> "Ask Claude to self-check. Append something like 'Before you finish, verify your answer against [test criteria].' This catches errors reliably, especially for coding and math. Claude Opus 5 is the exception: it verifies its own work well without explicit instruction, and verification instructions carried over from prompts tuned for earlier models can cause over-verification, adding tokens and latency. When migrating to Claude Opus 5, remove these instructions rather than rewriting them."

**Do not add this technique when the target is Claude Opus 5.** If the user's original prompt already contains a self-check or "double-check your answer" clause and the target is Opus 5, drop it during Step 3 and say so in one line — removing it is the documented fix, not rewording it. See `claude-opus-5.md` § 2.

Otherwise, when the prompt involves precise or verifiable outputs, add:

```
Before finalizing, verify your output against [specific criteria].
```

| Target | Technique 3 |
|---|---|
| Claude Opus 5 | **Omit.** Verification is default behavior; the instruction causes over-verification. |
| Claude Fable 5 / Mythos 5 | Keep — and on long runs prefer a fresh-context verifier subagent over self-critique (`claude-fable-5.md` § 8). |
| Claude Opus 4.8 and earlier, other runtimes | Keep as written. |
