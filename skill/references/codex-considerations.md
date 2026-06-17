# Codex-Specific Considerations

## When to Use

Load this file when the target runtime is OpenAI Codex (`gpt-5.5`, `gpt-5.4`, `gpt-5.4-mini`, and siblings). It covers behaviors that differ from Claude — preamble cadence that depends on the model version, explicit parallelization, strict `apply_patch` format, phase-aware output, dirty worktree handling, autonomy bias — and maps the 7-component framework to the official Codex Starter Prompt sections.

These guidelines are derived from OpenAI's official documentation and address behaviors particular to the Codex model family.

## Sources

- [OpenAI Codex Models](https://developers.openai.com/codex/models) — current model list and intended use cases
- [Codex Prompting Guide (GPT-5)](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) — recommended starter prompt and tuning guidance

## Recommended models (verified 2026-06-16)

| Model | Use case |
|---|---|
| `gpt-5.5` | Newest frontier model — **default choice in Codex** for complex coding, computer use, knowledge work, and research workflows ("For most tasks in Codex, start with `gpt-5.5`") |
| `gpt-5.4` | Flagship frontier model for professional work — strong coding, reasoning, tool use, and agentic workflows |
| `gpt-5.4-mini` | Fast, efficient mini model for responsive coding tasks and subagents — prioritize for lighter tasks when speed and cost matter |
| `gpt-5.3-codex-spark` | Text-only research preview optimized for near-instant, real-time iteration (ChatGPT Pro) |

> `gpt-5.2` and `gpt-5.3-codex` are deprecated for ChatGPT sign-in but may remain available via API-key authentication.

## Framework mapping

The 7-component framework applies directly to Codex. The table below maps each component to the equivalent section in OpenAI's recommended Starter Prompt:

| 7-component | Codex Starter Prompt section |
|---|---|
| Task | *Code Implementation* (engineering standards, objective, completeness) |
| Role | *Autonomy and Persistence* (the "autonomous senior engineer" persona framing) |
| Context | *Exploration and reading files* |
| Examples | *Editing constraints* (apply_patch format exemplars) |
| Output specification | *Presenting your work and final message* |
| Constraints | *Editing constraints* + *Autonomy and Persistence* |
| Structure | *General* (tool parallelization, section delimiters) |

> The current Starter Prompt also has dedicated *Plan tool*, *Special user requests*, and *Frontend tasks* sections. They do not map onto a single framework component — fold their guidance into Constraints (e.g., TODO/closure discipline) or Output specification (e.g., frontend UX expectations) when relevant.

## Codex-specific behaviors (differ from Claude)

1. **Preambles and upfront plans are now model-version dependent — do not blanket-suppress them.** For older Codex models the guide says to "remove all prompting for the model to communicate an upfront plan, preambles, or other status updates during the rollout, as this can cause the model to stop abruptly before the rollout is complete." But `gpt-5.3-codex` and newer *encourage* preambles on a cadence: aim for an update "every 1-3 execution steps; hard floor: at least within every 6 steps or 10 tool calls." So when migrating a prompt from Claude, decide based on the target Codex model: suppress narration on older models, but keep light cadence-based progress updates on `gpt-5.3-codex`+.

2. **Explicit parallelization.** Codex benefits from being told to batch independent tool calls. The guide's pattern is: plan all reads, issue them in a single parallel batch, analyze, repeat — "Use `multi_tool_use.parallel` to parallelize tool calls and only this" and "Never read files one-by-one unless logically unavoidable." Add a Constraint like: `"When multiple reads or commands are independent, issue them in a single parallel batch."`

3. **`apply_patch` format is strict.** Edits are expected in `apply_patch` form — either the Responses API built-in `apply_patch` tool type, or the freeform CFG grammar with `*** Begin Patch` / `*** End Patch` delimiters. If the Output specification references edits, exemplify the format or link to it — do not assume the model will use a freeform diff.

4. **Phase-aware output (`commentary` vs `final_answer`).** Assistant items carry a `phase` field: `phase: "commentary"` is preamble-style content, `phase: "final_answer"` is the closing message. "Correctly preserving `phase` on assistant items is required for `gpt-5.3-codex`." Consider this when the Output specification wants both running narration and a clean final artifact.

5. **Dirty git worktree handling.** Codex may encounter uncommitted changes. The guide is emphatic: "NEVER revert existing changes you did not make unless explicitly requested" and "NEVER use destructive commands like `git reset --hard` or `git checkout --` unless specifically requested"; if unexpected changes appear, "STOP IMMEDIATELY and ask the user how they would like to proceed." Add an explicit Constraint reflecting this when the task touches a working tree.

6. **Strong autonomy bias.** Codex is prompted to act as an "autonomous senior engineer" who proactively gathers context, plans, implements, tests, and refines without waiting, persisting "until the task is fully handled end-to-end within the current turn whenever feasible." When migrating a prompt from Claude, check your Constraints — guidance that encourages frequent confirmation on Claude may need softening for Codex, or vice versa.

7. **Structure format: Markdown headings or XML both work.** Unlike Claude, Codex has no strong preference. Its own final output uses plain text (the CLI handles styling), with optional headers, `-` bullets, and backticks for code/paths — no ANSI codes or deep nesting. Markdown headings (`## Task`, `## Output`) are often more natural in mixed human/agent prompts. See `framework.md` > Component 7 for the full matrix.

## Update procedure

When OpenAI releases a new Codex model or updates the prompting guide, refresh the "Recommended models" table (and the "verified" date), re-verify the mapping table against the current Starter Prompt structure, and re-check the preamble/phase guidance in point 1 and point 4 — those are the items most sensitive to model-version changes.
