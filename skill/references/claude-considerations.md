# Claude-Specific Considerations

## When to Use

Load this file when the target runtime is Anthropic Claude (Opus, Sonnet, Haiku, Fable, Mythos). It covers behaviors that the 7-component framework does not capture on its own — verbosity calibration, effort parameter, adaptive thinking, tone baseline, and other model-generation-specific tuning.

These guidelines are derived from Anthropic's official documentation and address behaviors particular to the Claude model family.

## How the official guidance is organized (as of 2026-06-16)

Anthropic split the prompting documentation into per-model pages. The main [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) page now holds only the techniques shared across all current models (general principles, output and formatting, tool use, thinking, agentic systems, migration). Model-specific behavior lives on its own page:

- [Prompting Claude Opus 4.8](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8) — the current flagship, default model string `claude-opus-4-8`.
- [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — covers Claude Fable 5 and Claude Mythos 5, the newest siblings (effort levels, instruction following, long-run progress claims, memory systems, the `reasoning_extraction` refusal category).

The section below tracks the current flagship (Opus 4.8). When a newer flagship ships, follow the [update procedure](#update-procedure).

## Claude Opus 4.8

Derived from the [Prompting Claude Opus 4.8](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8) page. Opus 4.8 "performs well out of the box on existing Claude Opus 4.7 prompts" — the points below focus on the behaviors that most often require tuning.

1. **Instructions are followed more literally.** "Claude Opus 4.8 interprets prompts literally and explicitly, particularly at lower effort levels. It does not silently generalize an instruction from one item to another, and it does not infer requests you didn't make." When you need broad application, state the scope in the Output specification or Constraints (e.g., "Apply this formatting to every section, not just the first one"). This literalism is a feature for API pipelines, structured extraction, and carefully tuned prompts where predictable behavior matters.

2. **Response length auto-calibrates to task complexity.** Opus 4.8 "calibrates response length to how complex it judges the task to be, rather than defaulting to a fixed verbosity" — shorter on simple lookups, longer on open-ended analysis. To reduce verbosity, prefer positive exemplars over negative instructions: `"Provide concise, focused responses. Skip non-essential context, and keep examples minimal."`

3. **Tone baseline is more direct and opinionated.** Opus 4.8 "tends toward a direct, opinionated style with minimal validation-forward phrasing and sparing emoji use." If the product relies on a warm voice, re-anchor it in the Output specification: `"Use a warm, collaborative tone. Acknowledge the user's framing before answering."`

4. **Effort is the primary dial — likely more important than on any prior Opus.** Start at `xhigh` for coding and agentic work; use a minimum of `high` for intelligence-sensitive use cases. `medium` is for cost-sensitive work; `low` for short, scoped, latency-sensitive tasks. `max` can help on intelligence-demanding tasks but shows diminishing returns and can overthink. Opus 4.8 respects effort strictly at the low end, so under-thinking is likely at `low`/`medium` on moderately complex tasks — raise effort rather than prompting around it. At `max`/`xhigh`, set an initial `max_tokens` of ~64k so the model has room to think and act across tool calls and subagents.

5. **Thinking is off by default; adaptive thinking is the path.** Thinking is off unless you explicitly set `thinking: {type: "adaptive"}`. Adaptive thinking calibrates depth from `effort` plus query complexity, and beats extended thinking in Anthropic's internal evals. Extended thinking with `budget_tokens` is deprecated — move budget control to `effort`. If the model over-thinks on simple queries (common with large system prompts), add: `"Thinking adds latency and should only be used when it will meaningfully improve answer quality — typically for problems that require multi-step reasoning. When in doubt, respond directly."`

6. **Favors reasoning over tool calls.** Opus 4.8 "has a tendency to favor reasoning over tool calls", which produces better results in most cases. Before adding aggressive tool prompts, raise `effort` to `high` or `xhigh` — that alone shows substantially more tool usage in agentic search and coding. Drop "If in doubt, use [tool]" style anti-laziness instructions; on this generation they cause over-triggering. If a specific tool is still under-used, describe explicitly when and how to use it.

7. **Fewer subagents and better progress updates by default.** Opus 4.8 spawns fewer subagents by default and gives more regular, higher-quality user-facing updates across long traces. Remove scaffolding that forced interim status ("After every 3 tool calls, summarize progress"). For subagents, be explicit only when you want fan-out, e.g.: `"Do not spawn a subagent for work you can complete directly in a single response. Spawn multiple subagents in the same turn when fanning out across items or reading multiple files."`

8. **XML tags remain the strongest formatting tool.** Claude parses XML tags unambiguously. Use them consistently for any prompt with more than two components. This is a Claude-specific recommendation — `framework.md` > Component 7 lists the format options valid across runtimes; on Claude, XML wins on both recall and precision in the official evals.

9. **Over-engineering in coding is still a risk.** Opus 4.8 inherits the 4.5/4.6 tendency to create extra files, add unnecessary abstractions, or build in flexibility that was not requested. Keep the Output specification tight on scope and, when needed, add:

   ```text
   Avoid over-engineering. Only make changes that are directly requested or clearly necessary.
   Don't add features, refactor unrelated code, add docstrings/comments to code you didn't change,
   or add error handling for scenarios that can't happen. The right amount of complexity is the
   minimum needed for the current task.
   ```

10. **Code review harnesses may need re-tuning.** Opus 4.8 is meaningfully better at finding bugs (higher recall and precision in internal evals) but follows filtering instructions like "only report high-severity issues" more faithfully, which can look like a recall regression — it investigates as deeply but reports fewer findings. Prefer: `"Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence at this stage — a separate verification step will do that. For each finding, include your confidence level and an estimated severity so a downstream filter can rank them."`

11. **Frontend design defaults are strong and persistent.** Opus 4.8 has a default house style: warm cream/off-white backgrounds (~`#F4F1EA`), serif display type (Georgia, Fraunces, Playfair), italic word-accents, and a terracotta/amber accent. It reads well for editorial/hospitality/portfolio briefs but feels off for dashboards, dev tools, fintech, healthcare, or enterprise apps, and it shows up in slide decks too. Generic overrides ("don't use cream", "make it clean") shift it to a different fixed palette rather than producing variety. Either specify a concrete alternative (palette hex values, typeface, radii, spacing) or ask the model to propose N distinct visual directions before building. Opus 4.8 needs less anti-"AI slop" prompting than earlier models.

12. **Interactive vs. autonomous coding differ in token use.** Opus 4.8 uses more tokens in interactive, multi-turn coding (it reasons more after each user turn), which improves long-horizon coherence but costs more. To maximize both performance and efficiency, use `xhigh`/`high` effort, add autonomous features (e.g., an auto mode), and specify the task, intent, and constraints fully in the first user turn rather than drip-feeding them across turns.

13. **Prefilled assistant responses are deprecated.** From Claude 4.6 onward, prefilling the last assistant turn is no longer supported (returns a 400). Migrate to: explicit system-prompt instructions, Structured Outputs, XML output tags, or user-turn continuation messages.

## Update procedure

When Anthropic releases a new flagship model generation (e.g., a Claude Opus 4.9 / 5.x, or a new Fable/Mythos generation):

1. Check whether the model gets its own `prompting-claude-<model>` page (the current pattern) or in-page guidance.
2. Add or rename the section above to track the new flagship, and refresh each numbered point against the new page.
3. Update the "How the official guidance is organized" block and the maintenance table in `framework.md`.
