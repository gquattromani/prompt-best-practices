# Claude Fable 5.x / Mythos 5.x — Tuning Notes

## When to Use

Load this file when the target runtime is Claude Fable 5.1 (`claude-fable-5-1`), Claude Mythos 5.1 (`claude-mythos-5-1`), Claude Fable 5 (`claude-fable-5`) or Claude Mythos 5 (`claude-mythos-5`) — the highest-capability tier, for the hardest long-running or ambiguous work. Fable 5.1 is the current model of the tier; Mythos 5.x shares the behavior and API surface of the matching Fable release. For the default Opus-tier target, see `claude-opus-5-5.md`. For the cross-model router and family-wide behaviors, see `claude-considerations.md`.

Derived from [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) and [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1) (both verified 2026-09-25, see `maintenance.md`) — every quotation below is taken verbatim from those pages; the instruction snippets shown in backticks are condensed adaptations of their sample prompts, not quotations. These notes summarize and comment on the pages; they are not a substitute for them. Fable 5 "takes on problems that were previously too complex, long-running, or ambiguous for prior models," and is aimed at end-to-end work that takes a person hours, days, or weeks.

## The headline: refactor, do not over-prescribe

This is the single most important point for a prompt-structuring skill. Anthropic is explicit: "Skills developed for prior models are often too prescriptive for Claude Fable 5 and can degrade output quality. Review and consider removing older instructions if default performance is better." Capability improvements at this level "are also a good prompt to re-evaluate which instructions, tools, and guardrails are still needed."

Concretely: include a framework component only when it changes the output. A tight `<task>` with a real success criterion plus a one-line brevity instruction outperforms a padded 7-section prompt. See `framework.md` > "Calibrating for frontier models".

Fable 5.1 does not reopen that question: "Your existing Claude Fable 5 prompts should perform well on Claude Fable 5.1 without changes, but a handful of behavioral differences are worth knowing about." The tuning points below hold for both releases; the next section lists the Fable 5.1 deltas.

## Tuning points (Fable 5 and 5.1)

1. **Steer with a brief instruction, not an enumeration.** "Instruction-following is improved enough that you can steer most behaviors with a brief instruction rather than enumerating each behavior by name." Un-steered — especially at higher effort — the model elaborates: surveying options it won't pursue, over-structured PR descriptions, comments narrating the next line. A short instruction is as effective as naming each pattern: `Lead with the outcome — the first sentence answers "what happened" or "what did you find". Keep output short by being selective, not by compressing into fragments or arrow chains.`

2. **Effort is the primary dial.** Effort is "the primary control for the trade-off between intelligence, latency, and cost." `high` is the default, `xhigh` for the most capability-sensitive workloads, `medium`/`low` for routine work. Reduce effort if a task completes but takes longer than necessary.

3. **Over-engineering is the risk at higher effort.** Higher effort buys rigorous output, but also unrequested tidying. Keep the Output specification tight on scope and, when needed, add: `Don't add features, refactors, or abstractions beyond what the task requires; do the simplest thing that works well; validate only at system boundaries.`

4. **Longer turns by default.** Individual requests on hard tasks "can run for many minutes at higher effort," and autonomous runs "can extend for hours". Adjust timeouts, streaming, and progress indicators before migrating. To stop overplanning on an ambiguous task: `When you have enough information to act, act. Do not re-litigate settled decisions or narrate options you will not pursue; give a recommendation, not a survey.`

5. **Ground progress claims in tool results.** On long autonomous runs this "nearly eliminated fabricated status reports" in Anthropic's testing: `Before reporting progress, audit each claim against a tool result from this session; say explicitly what is not yet verified.`

6. **State the boundaries.** The model can take unrequested actions: `When the user is describing a problem, asking a question, or thinking out loud, the deliverable is your assessment — report findings and stop.`

7. **Use parallel subagents freely.** Fable 5 "dispatches parallel subagents more readily than prior models" and is "significantly more dependable at dispatching and sustaining" them. **Note the divergence:** on Claude Opus 5 / 5.5 the correct move is the opposite — cap delegation (`claude-opus-5-5.md` > Inherited § 4).

8. **Make self-verification explicit on long runs.** "Separate, fresh-context verifier subagents tend to outperform self-critique." For long-running tasks: `Establish a method for checking your work as you build, and run it at intervals against the specification with subagents.` **Note the divergence:** this is exactly what must be *removed* for Claude Opus 5 / 5.5 (`claude-opus-5-5.md` > Inherited § 2).

9. **Give the reason, not only the request.** Fable 5 "tends to perform better when it understands the intent behind a request": `I'm working on [larger task] for [who]; they need [what the output enables]. With that in mind: [request].`

10. **Provide a memory surface.** Fable 5 "performs particularly well when it can record lessons from previous runs and reference them." A Markdown file is enough: one lesson per file, a one-line summary at the top, delete notes that turn out wrong.

11. **Readability after long runs.** When the final message is user-facing after a long session, ask for a re-grounding — outcome first, complete sentences, identifiers spelled out — not a continuation of the working thread.

12. **Consider a send-to-user tool** for long asynchronous agents: a client-side tool rendered verbatim delivers a deliverable mid-run. The model rarely calls it without an instruction pairing it to user-facing content only.

## What changes on Fable 5.1

1. **Re-run the effort sweep.** `high` stays the default and `max` joins the ladder; "Re-run the sweep even if you already ran one on Claude Fable 5: effort level names don't correspond to the same amount of thinking across models." At `medium`, "results roughly match Claude Fable 5 at lower cost". At `xhigh`/`max` the model can draft a long deliverable in its thinking and then write it again — run long deliverables at `high` unless a gain is measured, and leave `max_tokens` room.

2. **Ask for progress updates — the direction flipped.** The default "is to write fewer user-facing updates during long tool-calling turns than Claude Fable 5 does." Delete narration-suppressing lines written for earlier models ("hold all findings for the final response") before adding anything; then, where a human follows along, say when you want text: `Say in a line what you're about to do, give brief updates while you work, and close with a recap that stands on its own.` **Note the divergence:** Opus 5 needs narration damped (`claude-opus-5-5.md`); Fable 5.1 needs it requested.

3. **Finish the whole task.** On complex asynchronous workloads the model sometimes describes the next step instead of doing it, or asks permission for work already requested. Anthropic's autonomy block opens by telling the model the user is not watching and reversible in-scope actions need no permission; "The opening sentence, which tells the model the user isn't watching, carries much of the effect. Keep it as written." Use that block from the page for unattended runs, not for pair programming.

4. **Keep changes and tests to the request.** On open-ended features it may fix nearby code or commit more tests than needed; it "responds well to explicit instructions about what to leave out": `Report pre-existing bugs and unmentioned behavior as follow-ups instead of fixing them; implement the reading the wording most directly supports; commit tests only where asked or where the repository keeps them.`

5. **Remove anti-formatting rules.** "Claude Fable 5.1 leans the other way: it uses bold less and is less likely to reach for headers, lists, or quotation marks." A block written to hold down bullets on earlier models now suppresses structure the content needs. Replace it with a rule that says *when* a list helps. This is a *delete, not a rewrite* case.

6. **Name mannered prose.** Prose can run denser than Fable 5's. Defining the anti-pattern helps, and "The short version also tends to work": `Please remove all mannered prose.`

7. **Pin quotation behavior with one example.** When summarizing, the model is more likely to reproduce source passages unmarked. The documented fix is an example, not a rule: "add one complete example of a correct response to the system prompt: the user's request, the response, and a sentence explaining why the response is correct." This is the case where Component 4 earns its place on a frontier model.

8. **Nudge search at low effort.** At `low` the model is "more likely to answer from memory". Raise effort for the affected turns, or tell it that recognizing a name is not the same as knowing its current state, so unfamiliar or fast-moving names get searched.

9. **Ask for targeted edits.** It is "more likely than Claude Fable 5 to rewrite an entire text file rather than make a targeted edit": `When it will not affect the result, edit files surgically rather than rewriting them.`

10. **Harness changes, not prompt text.** Batch implied tool calls with a one-sentence nudge sent as a turn-scoped system message; keep the history append-only (thinking blocks are bound to the conversation that produced them); let the lead agent keep working while subagents run; tell client-side compaction exactly what to preserve.

## Adaptive thinking, refusals, and safeguards

Adaptive thinking is always on, with summarized-only thinking output; `budget_tokens` and `thinking: disabled` return a 400, and so do non-default `temperature`, `top_p`, or `top_k` values — move budget control to `effort`. The classifiers target offensive cybersecurity techniques, biology and life-sciences content, and extraction of summarized thinking. On Fable 5.1, false positives drop but still occur: ask "Are there any bugs in this program?" rather than whether it compiles, give documentation for lesser-known languages, and keep base64 out of tool output. Configure [fallback](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback); the permitted fallback targets for Fable 5.1 are Claude Opus 4.8 and Claude Opus 5.

**Do not instruct the model to reproduce its reasoning.** Prompts, skills, or harness instructions that tell it to echo, transcribe, or explain its internal reasoning as response text can trigger the `reasoning_extraction` refusal category. Audit for "show your thinking" / "explain your reasoning step by step" when migrating; read the structured `thinking` blocks instead.

## Recommended starting posture

Start at the top of your difficulty range: pick a task harder than you would assign to prior models, and have the model scope it, ask clarifying questions, then execute. Testing it only on simple workloads undersells its range — and, per the headline above, is where over-prescribed prompts do the most damage.
