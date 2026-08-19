# Claude Fable 5 / Mythos 5 — Tuning Notes

## When to Use

Load this file when the target runtime is Claude Fable 5 (`claude-fable-5`) or Claude Mythos 5 (`claude-mythos-5`) — the highest-capability tier, for the hardest long-running or ambiguous work. For the default Opus-tier target, see `claude-opus-5.md`. For the cross-model router and family-wide behaviors, see `claude-considerations.md`.

Derived from [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) (verified 2026-08-03) — every quotation below is taken verbatim from that page; the instruction snippets shown in backticks are condensed adaptations of its sample prompts, not quotations. These notes summarize and comment on that page; they are not a substitute for it. Fable 5 "takes on problems that were previously too complex, long-running, or ambiguous for prior models," and is aimed at end-to-end work that takes a person hours, days, or weeks. Mythos 5 shares the same behavior and API surface.

## The headline: refactor, do not over-prescribe

This is the single most important point for a prompt-structuring skill. Anthropic is explicit: "Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality. Review and consider removing older instructions if default performance is better." Capability improvements at this level "are also a good prompt to re-evaluate which instructions, tools, and guardrails are still needed."

Concretely: include a framework component only when it changes the output. A tight `<task>` with a real success criterion plus a one-line brevity instruction outperforms a padded 7-section prompt. See `framework.md` > "Calibrating for frontier models".

## Tuning points

1. **Steer with a brief instruction, not an enumeration.** "Instruction-following is improved enough that you can steer most behaviors with a brief instruction rather than enumerating each behavior by name." Un-steered — especially at higher effort — Fable 5 elaborates: surveying options it won't pursue, over-structured PR descriptions, comments narrating the next line. A short brevity instruction is as effective as naming each pattern:

   ```text
   Lead with the outcome. Your first sentence after finishing should answer "what happened" or "what did
   you find" — the TLDR the user would ask for. Supporting detail comes after. Keep output short by being
   selective about what you include, not by compressing into fragments, abbreviations, or arrow chains.
   ```

   The same applies to checkpoints in long workflows — no need to enumerate every case: `"Pause for the user only when the work genuinely requires them: a destructive or irreversible action, a real scope change, or input that only they can provide. If you hit one of these, ask and end the turn, rather than ending on a promise."`

2. **Effort is the primary dial.** Effort is "the primary control for the trade-off between intelligence, latency, and cost." `high` is the default, `xhigh` for the most capability-sensitive workloads, `medium`/`low` for routine work — lower settings "still perform well and often exceed `xhigh` performance on prior models." Reduce effort if a task completes but takes longer than necessary.

3. **Over-engineering is the risk at higher effort.** Higher effort buys excellent verification behavior and rigorous output, but also unrequested tidying. Keep the Output specification tight on scope and, when needed, add:

   ```text
   Don't add features, refactor, or introduce abstractions beyond what the task requires. A bug fix
   doesn't need surrounding cleanup. Don't design for hypothetical future requirements: do the simplest
   thing that works well. Don't add error handling, fallbacks, or validation for scenarios that cannot
   happen — only validate at system boundaries (user input, external APIs).
   ```

4. **Longer turns by default.** Individual requests on hard tasks "can run for many minutes at higher effort," and autonomous runs "can extend for hours" — one of the largest shifts teams encounter. Adjust timeouts, streaming, and progress indicators before migrating; prefer async check-ins over blocking. To stop it overplanning on an ambiguous task: `"When you have enough information to act, act. Do not re-derive facts already established, re-litigate a decision already made, or narrate options you will not pursue. If you are weighing a choice, give a recommendation, not an exhaustive survey. This does not apply to thinking blocks."`

5. **Ground progress claims in tool results.** On long autonomous runs this "nearly eliminated fabricated status reports" in Anthropic's testing: `"Before reporting progress, audit each claim against a tool result from this session. Only report work you can point to evidence for; if something is not yet verified, say so explicitly."`

6. **State the boundaries.** Fable 5 can take unrequested actions (drafting an email nobody asked for, creating defensive git backups): `"When the user is describing a problem, asking a question, or thinking out loud rather than requesting a change, the deliverable is your assessment. Report findings and stop. Don't apply a fix until they ask."`

7. **Use parallel subagents freely.** Fable 5 "dispatches parallel subagents more readily than prior models" and is "significantly more dependable at dispatching and sustaining" them. Delegate often, say when delegation is appropriate, and prefer asynchronous orchestrator-subagent communication over blocking on each return. **Note the divergence:** on Claude Opus 5 the correct move is the opposite — cap delegation (`claude-opus-5.md` § 4).

8. **Make self-verification explicit on long runs.** "Separate, fresh-context verifier subagents tend to outperform self-critique." For long-running tasks: `"Establish a method for checking your own work as you build. Run it every [interval], verifying against the specification with subagents."` **Note the divergence:** this instruction is exactly what must be *removed* for Claude Opus 5, which over-verifies when told to verify (`claude-opus-5.md` § 2).

9. **Give the reason, not only the request.** Fable 5 "tends to perform better when it understands the intent behind a request": `"I'm working on [larger task] for [who]. They need [what the output enables]. With that in mind: [request]."` Same principle as Component 6 (constraints with motivation), applied to the task framing as a whole.

10. **Provide a memory surface.** Fable 5 "performs particularly well when it can record lessons from previous runs and reference them." A Markdown file is enough: one lesson per file, a one-line summary at the top, record corrections and confirmed approaches, delete notes that turn out wrong.

11. **Add a readability addendum for long runs.** In extended agentic conversations Fable 5 can produce dense arrow-chain shorthand or reference thinking the user never saw. When the output is user-facing after a long run, instruct it to write the final message as a re-grounding — outcome first, complete sentences, identifiers spelled out — not a continuation of its working thread.

12. **Rare: early stopping and context-budget concern.** Deep into a long session it may end a turn on a statement of intent without the tool call, or suggest starting a new session (most often when the harness shows a remaining-token countdown). For autonomous pipelines, add a system reminder that the user is not watching, that reversible in-scope actions proceed without asking, and that a turn ending on a promise should instead do the work. Avoid surfacing explicit context-budget counts.

13. **Consider a send-to-user tool.** For long asynchronous agents, a client-side tool whose input you render verbatim delivers a deliverable or a direct answer mid-run without ending the turn. Defining it is not enough — Fable 5 rarely calls it without an instruction pairing it to user-facing content only.

## Adaptive thinking, refusals, and safeguards

Fable 5 supports adaptive thinking only, with summarized-only thinking output; extended thinking with `budget_tokens` is gone — move budget control to `effort`. It runs classifiers targeting offensive cybersecurity techniques, biology and life-sciences content, and extraction of its summarized thinking; benign work in those domains may also trip them. Configure [server- or client-side fallback](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback) to Claude Opus 4.8 for declined requests.

**Do not instruct Fable 5 to reproduce its reasoning.** Prompts, skills, or harness instructions that tell it to echo, transcribe, or explain its internal reasoning as response text can trigger the `reasoning_extraction` refusal category and cause elevated fallbacks. Audit for "show your thinking" / "explain your reasoning step by step" when migrating. If you need reasoning visibility, read the structured `thinking` blocks instead.

## Recommended starting posture

Start at the top of your difficulty range: pick a task harder than you would assign to prior models, and have Fable 5 scope it, ask clarifying questions, then execute. Testing it only on simple workloads undersells its range — and, per the headline above, is where over-prescribed prompts do the most damage.
