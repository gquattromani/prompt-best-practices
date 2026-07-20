# Grounding and Accuracy Techniques

## When to Use

Load this file when the prompt involves accuracy-critical work: reading an existing codebase, quoting from documents, verifying outputs against constraints, or any task where hallucinations would cause real harm. The three techniques below can be added as Constraints (Component 6) or woven into the task instruction.

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

### 3. Self-check before finalizing

From [Leverage thinking & interleaved thinking capabilities](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#leverage-thinking--interleaved-thinking-capabilities):

> "Ask Claude to self-check. Append something like 'Before you finish, verify your answer against [test criteria].' This catches errors reliably, especially for coding and math."

When the prompt involves precise or verifiable outputs, add:

```
Before finalizing, verify your output against [specific criteria].
```
