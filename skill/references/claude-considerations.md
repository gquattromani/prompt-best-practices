# Claude-Specific Considerations

## When to Use

Load this file when the target runtime is Anthropic Claude (Opus, Sonnet, Haiku). It covers behaviors that the 7-component framework does not capture on its own — verbosity calibration, effort parameter, adaptive thinking, tone baseline, and other model-generation-specific tuning.

These guidelines are derived from Anthropic's official documentation and address behaviors particular to the Claude model family.

## Claude 4.7

Derived from the [Prompting Claude Opus 4.7](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#prompting-claude-opus-4-7) section of the official guide. Opus 4.7 is backwards-compatible with Opus 4.6 prompts — the points below focus on the behaviors that benefit most from tuning.

1. **Instructions are followed more literally.** "Opus 4.7 interprets prompts more literally and explicitly than Opus 4.6 ... it will not silently generalize an instruction from one item to another, and it will not infer requests that were not explicitly made." When you need broad application, state the scope in the Output specification or Constraints (e.g., "Apply this formatting to every section, not just the first").

2. **Response length auto-calibrates to task complexity.** Opus 4.7 picks its own verbosity instead of running at a fixed level. Avoid forcing verbosity; to reduce it, prefer positive exemplars over negative instructions: `"Provide concise, focused responses. Skip non-essential context, and keep examples minimal."`

3. **Tone baseline is more direct and opinionated.** "Claude Opus 4.7 is more direct and opinionated, with less validation-forward phrasing and fewer emoji than Opus 4.6." If the product relies on a warm voice, re-anchor it in the Output specification: `"Use a warm, collaborative tone. Acknowledge the user's framing before answering."`

4. **Effort parameter and adaptive thinking are the primary dials.** Opus 4.7 adds the `xhigh` level (recommended default for coding and agentic work) and respects `low`/`medium` strictly — under-thinking at `low` is likely on moderately complex tasks. Prefer `thinking: {type: "adaptive"}` over `extended_thinking` with `budget_tokens`. At `max`/`xhigh` set an initial `max_tokens` of ~64k so the model has room to plan and act. If the model over-thinks on simple queries, add: `"Thinking adds latency and should only be used when it will meaningfully improve answer quality. When in doubt, respond directly."`

5. **Less tool use and less subagent spawning by default.** Opus 4.7 leans on reasoning more than 4.6. Before adding more aggressive tool prompts, raise `effort` to `high` or `xhigh` — that alone increases tool usage noticeably. Drop "If in doubt, use [tool]" style anti-laziness instructions; they now cause over-triggering. For subagents, be explicit only when you actually want parallel fan-out.

6. **XML tags remain the strongest formatting tool.** Claude parses XML tags unambiguously. Use them consistently for any prompt with more than two components. This is a Claude-specific recommendation — `framework.md` > Component 7 lists the format options valid across runtimes; on Claude, XML wins on both recall and precision in the official evals.

7. **Over-engineering in coding is still a risk.** Opus 4.7 inherits 4.6's tendency to create extra files, add unnecessary abstractions, or build in flexibility that was not requested. Keep the Output specification tight on scope and, when needed, add:

   ```text
   Avoid over-engineering. Only make changes that are directly requested or clearly necessary.
   Do not add features, refactor unrelated code, or introduce abstractions for hypothetical future requirements.
   ```

8. **Code review harnesses may need re-tuning.** Opus 4.7 finds more bugs (+11pp recall on one eval) but follows filtering instructions like "report only high-severity issues" more faithfully, which can look like a regression. Prefer: `"Report every issue you find, including low-severity or uncertain ones. A downstream filter will rank them. Include confidence and severity for each finding."`

9. **Frontend design defaults are stronger.** Opus 4.7 has a persistent house style (cream/off-white backgrounds, serif display type, terracotta/amber accents). Generic overrides ("don't use cream", "make it clean") shift it to a different fixed palette. Either specify a concrete alternative (palette hex values, typeface, radii, spacing) or ask the model to propose N distinct visual directions before building.

10. **Prefilled assistant responses are deprecated.** From Claude 4.6 onward, prefilling the last assistant turn is no longer supported (Mythos Preview returns a 400). Migrate to: explicit system-prompt instructions, Structured Outputs, XML output tags, or user-turn continuation messages.

## Update procedure

When Anthropic releases a new model generation (e.g., Claude 4.8, 5.x), update this file with the new guidance and rename the relevant section accordingly.
