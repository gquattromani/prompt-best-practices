# Codex-Specific Considerations

## When to Use

Load this file when the target runtime is OpenAI Codex (the GPT-5.6 family: `gpt-5.6-sol`, `gpt-5.6-terra`, `gpt-5.6-luna`, and siblings). It covers behaviors that differ from Claude — the outcome-first / lean-prompt principle, the `reasoning.effort` ladder, verbosity control, autonomy and approval boundaries, multi-agent and programmatic tool calling, strict `apply_patch` format — and maps the 7-component framework onto OpenAI's current guidance.

These guidelines are derived from OpenAI's official documentation and address behaviors particular to the current Codex model family.

## Sources

- [Model guidance — GPT-5.6](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6) — the official prompting guidance for the GPT-5.6 family
- [Codex / GPT models overview](https://learn.chatgpt.com/docs/models) — current model list and intended use cases

## Recommended models (verified 2026-07-20)

| Model | Use case |
|---|---|
| `gpt-5.6-sol` | **Frontier flagship — default choice in Codex** for complex coding, computer use, research, and cybersecurity ("Sol with medium reasoning is the recommended starting point"). Reachable via the `gpt-5.6` alias. |
| `gpt-5.6-terra` | Balanced everyday model — "performance competitive with GPT-5.5 at a lower cost". Prefer for standard work where cost matters. |
| `gpt-5.6-luna` | Fast, affordable, high-volume workloads with well-defined outcomes — good for subagents and lighter tasks. |
| `gpt-5.5` | Previous-generation frontier model, still available. |
| `gpt-5.3-codex-spark` | Text-only research preview optimized for near-instant, real-time coding iteration (ChatGPT Pro). |

> `gpt-5.2` and `gpt-5.3-codex` are deprecated for ChatGPT sign-in; replace them in existing scripts and configs.

## The headline principle: outcome-first, leaner prompts

This is the biggest change in the GPT-5.6 family and it directly shapes how the 7-component framework should be applied. From the official guidance:

> "Removing repeated instructions and examples and simplifying tool descriptions can improve task performance and token efficiency."

In OpenAI's internal coding-agent tests, leaner system prompts improved evaluation scores by **~10-15%** while cutting tokens by **41-66%** and cost by **33-67%**. The model infers underlying user goals better than prior generations, so verbose, instruction-heavy prompts now act as noise.

**What to remove** when building or migrating a prompt for GPT-5.6:
- Instructions stated more than once.
- Examples that do not change behavior (keep only those encoding a real product requirement).
- Tool descriptions beyond a concise, precise essential.
- Generic, redundant approval language ("ask first", "wait for approval") on safe actions.

**What to keep:**
- Domain context and hard constraints.
- Approval boundaries and success criteria.
- One clear statement per instruction.
- Only task-relevant tools.

For the 7-component framework this means: **Task** (with a real success criterion), **Output specification**, and **Constraints** (with motivation) carry almost all the weight on GPT-5.6. **Examples** should be included only when a single example changes the output — otherwise drop them. This is the same direction Claude Fable 5 took (see `claude-considerations.md` § 1); `framework.md` > "Calibrating for frontier models" is the shared statement of the principle.

## reasoning.effort

Supported settings: `none`, `low`, `medium`, `high`, `xhigh`, `max`. Default is `medium`.

- Migrating from GPT-5.5 / GPT-5.4: "preserve your current reasoning effort as the baseline, then compare one level lower" — the GPT-5.6 family often matches prior quality at lower effort.
- `low` for latency-sensitive work; `medium` as the balanced starting point; `high`/`xhigh` when extra reasoning produces measured quality gains; `max` for the hardest, quality-first workloads only.

## Verbosity control

Use the `text.verbosity` parameter (`low` / `medium` / `high`) for the default detail level, and specify task-specific length in the prompt. For short answers, the guidance recommends: `"Lead with the conclusion. Include the evidence needed to support it, any material caveat, and the next action."` This maps onto framework Component 5 (Output specification).

## Autonomy and approval boundaries

Rather than sprinkling "ask first" across the prompt, define once what each request authorizes:

> "Define what level of action each request authorizes so the model can continue safe, in-scope work without unnecessary pauses while stopping before external, destructive, costly, or scope-expanding actions."

Fold this into Component 6 (Constraints) — one boundary statement, not per-action nagging.

## Tool use

- **Multi-agent [beta].** A GPT-5.6 instance can coordinate multiple subagents in parallel and synthesize their results — useful for complex tasks that divide cleanly into independent workstreams.
- **Programmatic Tool Calling (PTC).** Best for bounded workflows that process multiple tool results into a smaller structured output (filtering, joining, ranking, aggregation). State explicitly which stage uses PTC, the eligible tools, the output schema, and concurrency limits. Direct calling stays preferable when one call suffices, intermediate outputs are small, each result changes the next decision, approval is required, or citations must be preserved.
- **`apply_patch` format is strict.** Edits are expected in `apply_patch` form — the Responses API built-in `apply_patch` tool type, or the freeform grammar with `*** Begin Patch` / `*** End Patch` delimiters. If the Output specification references edits, exemplify or link the format; do not assume a freeform diff.

## Framework mapping

The 7-component framework applies to Codex, but apply it through the lean-prompt lens above — include a component only when it changes behavior.

| 7-component | Where it lands on GPT-5.6 |
|---|---|
| Task | Objective + success criteria — the load-bearing component. |
| Role | Usually implicit; a one-line persona at most. Codex already assumes an autonomous senior engineer. |
| Context | Domain context and files to read; keep it to what the task needs. |
| Examples | Include only if a single example changes the output — otherwise omit. |
| Output specification | Final-message shape, verbosity, `apply_patch` format for edits. |
| Constraints | Hard constraints, approval boundaries, dirty-worktree rules — stated once, with motivation. |
| Structure | Markdown headings or XML both work (see below). |

## Other Codex-specific behaviors

1. **Dirty git worktree handling.** Codex may encounter uncommitted changes. The guidance is emphatic: never revert changes you did not make and never use destructive commands (`git reset --hard`, `git checkout --`) unless explicitly requested; if unexpected changes appear, stop and ask. State this as a Constraint when the task touches a working tree.
2. **Persisted reasoning and Pro mode.** `reasoning.context` reuses reasoning across turns for multi-turn quality; `reasoning.mode: "pro"` spends more model effort for reliability on hard tasks (single final answer, independent from `reasoning.effort`). Neither needs prompt changes, but they affect how much scaffolding a prompt needs — less, generally.
3. **Explicit prompt caching.** Mark reusable prefixes to cache them (cache writes cost 1.25x; track `cached_tokens` / `cache_write_tokens`). Relevant when a structured prompt template is reused across many calls.
4. **Structure format: Markdown headings or XML both work.** Unlike Claude, Codex has no strong preference. Its own final output is plain text with optional headers, `-` bullets, and backticks for code/paths — no deep nesting. Markdown headings (`## Task`, `## Output`) are often more natural in mixed human/agent prompts. See `framework.md` > Component 7.
5. **Safety classifiers.** Real-time cyber- and biology-misuse classifiers may block or pause generation mid-stream on dual-use content. Send a stable, privacy-preserving `safety_identifier` for individual end-user applications.

## Update procedure

When OpenAI releases a new model or updates the prompting guidance:

1. Refresh the "Recommended models" table and the "verified" date.
2. Re-check the outcome-first / lean-prompt principle — it is the load-bearing section and the one most likely to be extended (new "what to remove" items).
3. Re-verify the `reasoning.effort` ladder and default, the framework-mapping table, and the tool-use section (multi-agent / PTC) against the current guidance.
