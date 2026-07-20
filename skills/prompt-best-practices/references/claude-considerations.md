# Claude-Specific Considerations

## When to Use

Load this file when the target runtime is Anthropic Claude (Fable, Mythos, Opus, Sonnet, Haiku). It covers behaviors that the 7-component framework does not capture on its own — effort calibration, adaptive thinking, instruction-following literalness, long-run behavior, subagents, memory, and other model-generation-specific tuning.

These guidelines are derived from Anthropic's official documentation and address behaviors particular to the Claude model family.

## How the official guidance is organized (as of 2026-07-20)

Anthropic keeps the prompting documentation split into per-model pages. The main [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) page is organized in three parts: **model-specific guidance** first, **techniques for all current models** (general principles, output and formatting, tool use, thinking, agentic systems) after, and **migration considerations** last. Model-specific behavior lives on its own page:

- [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) — the current flagship reference, covering Claude Fable 5 and Claude Mythos 5 (`claude-fable-5`, `claude-mythos-5`).
- [Prompting Claude Sonnet 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-sonnet-5) — the balanced workhorse.
- [Prompting Claude Opus 4.8](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8) — the prior flagship, now the recommended **fallback target** when Fable 5 declines a request.

The primary section below tracks the current flagship (Fable 5). When a newer flagship ships, follow the [update procedure](#update-procedure).

## Claude Fable 5

Derived from the [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) page. Fable 5 is built for "problems that were previously too complex, long-running, or ambiguous for prior models" — end-to-end work that takes a person hours to weeks. It also performs reliably on straightforward tasks. The points below focus on the behaviors that most often require tuning relative to Opus 4.8.

1. **Refactor prompts and skills — do not over-prescribe.** This is the single most important shift for a prompt-structuring skill. Anthropic is explicit: "Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality. Review and consider removing older instructions if default performance is better." Capability improvements are "a good prompt to re-evaluate which instructions, tools, and guardrails are still needed." Concretely: include a framework component only when it changes the output. A tight `<task>` with a real success criterion plus a one-line brevity instruction now outperforms a padded 7-section prompt. See `framework.md` > "Calibrating for frontier models".

2. **Strong instruction following — steer with a brief instruction, not an enumeration.** "Instruction-following is improved enough that you can steer most behaviors with a brief instruction rather than enumerating each behavior by name." Un-steered, Fable 5 can over-elaborate at higher effort (surveying options it won't pursue, over-structured PR descriptions, comments narrating the next line). A short brevity instruction is as effective as listing each pattern:

   ```text
   Lead with the outcome. Your first sentence after finishing should answer "what happened" or
   "what did you find" — the TLDR the user would ask for. Supporting detail comes after. Keep output
   short by being selective about what you include, not by compressing into fragments, abbreviations,
   or arrow chains.
   ```

3. **Effort is the primary dial.** Effort is "the primary control for the trade-off between intelligence, latency, and cost." Use `high` as the default for most tasks, `xhigh` for the most capability-sensitive workloads, `medium`/`low` for routine work. Lower effort settings on Fable 5 "still perform well and often exceed `xhigh` performance on prior models." Reduce effort if a task completes but takes longer than necessary. At higher effort Fable 5 can gather context and deliberate beyond what the task needs — pair high effort with the anti-over-engineering block in point 9.

4. **Adaptive thinking only; no extended-thinking budgets.** Fable 5 supports adaptive thinking with summarized-only thinking output. Extended thinking with `budget_tokens` is gone — move budget control to `effort`. There is a new `refusal` stop reason; configure [server- or client-side fallback](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback) to Claude Opus 4.8 for declined requests.

5. **Longer turns by default.** Individual requests on hard tasks "can run for many minutes at higher effort," and autonomous runs "can extend for hours." Adjust client timeouts, streaming, and progress indicators before migrating; prefer async harnesses (scheduled check-ins) over blocking. To keep Fable 5 from overplanning on ambiguous tasks:

   ```text
   When you have enough information to act, act. Do not re-derive facts already established, re-litigate
   a decision already made, or narrate options you will not pursue. If you are weighing a choice, give a
   recommendation, not an exhaustive survey. This does not apply to thinking blocks.
   ```

6. **Ground progress claims in tool results.** On long autonomous runs, instruct Fable 5 to audit each claim against an actual tool result — in Anthropic's testing this "nearly eliminated fabricated status reports." Add: `"Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so."`

7. **State the boundaries.** Fable 5 can occasionally take unrequested actions (drafting an email nobody asked for, creating defensive git backups). Define what it should and should not do: `"When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report findings and stop. Don't apply a fix until they ask."`

8. **Parallel subagents by default.** Fable 5 "dispatches parallel subagents more readily than prior models" and is "significantly more dependable at dispatching and sustaining parallel subagents." Use subagents frequently, give explicit guidance on when delegation is appropriate, and prefer asynchronous orchestrator-subagent communication over blocking. For self-verification, "separate, fresh-context verifier subagents tend to outperform self-critique."

9. **Over-engineering / unrequested tidying is a risk at higher effort.** Keep the Output specification tight on scope and, when needed, add:

   ```text
   Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix
   doesn't need surrounding cleanup. Don't design for hypothetical future requirements: do the simplest
   thing that works well. Don't add error handling, fallbacks, or validation for scenarios that cannot
   happen — only validate at system boundaries (user input, external APIs).
   ```

10. **Do not instruct Claude to reproduce its reasoning.** Prompts, skills, or harness instructions that tell the model to echo, transcribe, or explain its internal reasoning as response text can trigger the `reasoning_extraction` refusal category and cause elevated fallbacks to Opus 4.8. Audit for "show your thinking" / "explain your reasoning step by step" instructions when migrating. If you need reasoning visibility, read the structured `thinking` blocks from adaptive thinking; for progress during long runs, use a send-to-user tool.

11. **Give the reason, not only the request.** Fable 5 "performs better when it understands the intent behind a request." Provide the why, especially for long-running agents: `"I'm working on [larger task] for [who]. They need [what the output enables]. With that in mind: [request]."` This is the same principle as framework Component 6 (Constraints with motivation), now applying to the task framing as a whole.

12. **Memory helps.** Fable 5 "performs particularly well when it can record lessons from previous runs and reference them." Give it a place to write notes (a Markdown file is enough): one lesson per file, a one-line summary at the top, record corrections and confirmed approaches, delete notes that turn out wrong.

13. **Construct a memory system / readability addendum for long runs.** In extended agentic conversations, Fable 5 can produce dense arrow-chain shorthand or reference thinking the user never saw. When the output is user-facing after a long run, add a communication-style instruction telling it to write the final message as a re-grounding (outcome first, complete sentences, spell out identifiers), not a continuation of its working thread.

14. **Safety classifiers.** Fable 5 runs classifiers targeting offensive cybersecurity techniques (exploits, malware, attack tooling), biology/life-sciences content, and extraction of summarized thinking. Benign cyber and beneficial life-sciences work may also trip them. For those domains, configure fallback to Opus 4.8, which does not run these classifiers as aggressively.

15. **XML tags remain the strongest formatting tool on Claude.** Claude parses XML tags unambiguously. Use them consistently for any prompt with more than two components. This is a Claude-specific recommendation — `framework.md` > Component 7 lists the runtime-agnostic format options.

16. **Prefilled assistant responses remain deprecated.** From Claude 4.6 onward, prefilling the last assistant turn is unsupported (returns a 400). Migrate to explicit system-prompt instructions, Structured Outputs, XML output tags, or user-turn continuation messages.

## Claude Opus 4.8 (fallback target)

Opus 4.8 remains a strong, widely-deployed model and is the recommended fallback when Fable 5 declines a request (see point 4). Its dedicated [Prompting Claude Opus 4.8](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-4-8) page still applies. The most load-bearing differences from Fable 5: Opus 4.8 interprets instructions more literally at low effort (state scope explicitly), starts coding/agentic work at `xhigh`, and shares the same over-engineering risk (point 9) and prefill deprecation (point 16). When you author a prompt for a Fable-5-with-Opus-4.8-fallback pipeline, keep it valid for both — the 7-component framework is, by construction.

## Update procedure

When Anthropic releases a new flagship model generation (e.g. a Fable/Mythos 6, or a new Opus/Sonnet generation):

1. Check whether the model gets its own `prompting-claude-<model>` page (the current pattern) or in-page guidance.
2. Add or rename the primary section above to track the new flagship, and refresh each numbered point against the new page. Demote the outgoing flagship to a "fallback target" section if it is still the recommended fallback.
3. Update the "How the official guidance is organized" block and the maintenance table in `framework.md`.
4. Re-check point 1 (over-prescription) against the new page — frontier models keep raising the bar on default performance, which usually means *fewer* framework components add value, not more.
