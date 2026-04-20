# Codex-Specific Considerations

## When to Use

Load this file when the target runtime is OpenAI Codex (`gpt-5.4`, `gpt-5.3-codex`, and siblings). It covers behaviors that differ from Claude — no upfront plans, explicit parallelization, strict `apply_patch` format, phase-aware output, dirty worktree handling — and maps the 7-component framework to the official Codex Starter Prompt sections.

These guidelines are derived from OpenAI's official documentation and address behaviors particular to the Codex model family.

## Sources

- [OpenAI Codex Models](https://developers.openai.com/codex/models) — current model list and intended use cases
- [Codex Prompting Guide (GPT-5)](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide) — recommended starter prompt and tuning guidance

## Recommended models (verified 2026-04-20)

| Model | Use case |
|---|---|
| `gpt-5.4` | Flagship model for professional coding — default choice for complex software engineering |
| `gpt-5.4-mini` | Fast, efficient model for subagents and responsive coding tasks |
| `gpt-5.3-codex` | Previous-generation specialized coding model — still available |
| `gpt-5.3-codex-spark` | Real-time preview optimized for near-instant iteration (ChatGPT Pro) |

## Framework mapping

The 7-component framework applies directly to Codex. The table below maps each component to the equivalent section in OpenAI's recommended Starter Prompt:

| 7-component | Codex Starter Prompt section |
|---|---|
| Task | *Code Implementation* (objective and completeness) |
| Role | *Preambles & Personality* |
| Context | *Exploration and reading files* |
| Examples | *Editing constraints* (apply_patch format exemplars) |
| Output specification | *Presenting your work* |
| Constraints | *Editing constraints* + *Autonomy and Persistence* |
| Structure | *General* (tool parallelization, section delimiters) |

## Codex-specific behaviors (differ from Claude)

1. **Avoid upfront plans and preambles.** The official guide explicitly says: "Avoid prompting for the model to communicate an upfront plan, preambles, or other status updates during the rollout." This is the opposite of some Claude patterns where asking the model to outline its approach helps. Keep the Output specification focused on the final deliverable, not on narration.

2. **Explicit parallelization.** Codex benefits from being told to batch independent tool calls (`multi_tool_use.parallel`). Unlike recent Claude models, auto-parallelism is weaker — add a Constraint like: `"When multiple reads or commands are independent, issue them in a single parallel batch."`

3. **`apply_patch` format is strict.** Edits are expected in `apply_patch` blocks with precise structure. If the Output specification references edits, exemplify the format or link to it — do not assume the model will use a freeform diff.

4. **Phase-aware output.** `gpt-5.3-codex` supports a `commentary` vs `final_answer` phase split. Consider this when the Output specification wants both reasoning and a clean final artifact.

5. **Dirty git worktree handling.** Codex may touch uncommitted changes. Add an explicit Constraint: `"Never revert or overwrite uncommitted user changes. Investigate unfamiliar files before acting on them."`

6. **Strong autonomy bias.** Codex prompts typically push for aggressive autonomy ("do not stop to ask for clarifications"). When migrating a prompt from Claude, check your Constraints — guidance that encourages frequent confirmation on Claude may need softening for Codex, or vice versa.

7. **Structure format: Markdown headings or XML both work.** Unlike Claude, Codex has no strong preference between Markdown headings (`## Task`, `## Output`) and XML tags. Markdown headings are often more natural in mixed human/agent prompts. See `framework.md` > Component 7 for the full matrix.

## Update procedure

When OpenAI releases a new Codex model or updates the prompting guide, refresh the "Recommended models" table and re-verify the mapping table against the current Starter Prompt structure.
