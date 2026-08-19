# Claude Opus 5 — Tuning Notes

## When to Use

Load this file when the target runtime is Claude Opus 5 (`claude-opus-5`) — the default target for most Claude work, including Claude Code. For the cross-model router and the behaviors shared by the whole family, see `claude-considerations.md`. For the highest-capability tier, see `claude-fable-5.md`.

Derived from [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) (verified 2026-08-03; quotations re-checked 2026-08-19, see `maintenance.md`) — text in double quotes is quoted verbatim from that page; the fenced instruction blocks are condensed adaptations of its sample prompts, not quotations. These notes summarize and comment on that page; they are not a substitute for it. Claude Opus 5 is built for complex agentic coding and enterprise work, with particular strength on long-horizon agentic tasks.

## Start from the prompt you already have

Anthropic is explicit: Opus 5 "performs well out of the box on existing Claude Opus 4.8 prompts." A migration is not a rewrite — the points below are the behaviors that most often need tuning, plus two API-level changes: thinking is **on by default**, and disabling thinking is accepted only at `effort` `high` or below.

## The eight tuning points

1. **Prompt for conciseness — `effort` will not do it.** Default user-facing responses run longer than on prior Opus models, and "the effort parameter controls how much the model thinks rather than how much it says: lowering effort can reduce thinking volume without reliably shortening the visible response." This makes Component 5 (Output specification) load-bearing on Opus 5 even for quick-tier prompts:

   ```text
   Keep the response focused and brief: spend it on the answer, keep caveats short, and default to a
   high-level summary unless depth was asked for.
   ```

   In a long prompt, pair it with a short reminder near the end: `<tone_preference>Keep outputs reasonably concise.</tone_preference>`

2. **Delete verification instructions — do not rewrite them.** Opus 5 verifies its own work without being told. Explicit verification instructions — a final verification step on every non-trivial task, a subagent whose job is to double-check — are to be removed rather than reworded, because they "cause over-verification on Claude Opus 5, and removing them reduces wasted tokens with no loss in quality." The same applies to re-check phrasing ("double-check your answer," "re-verify before responding") and to legacy harness scaffolding that adds a separate verification step.

   For this skill: when the target is Opus 5, **skip technique 3 (self-check) in `grounding-techniques.md`** and drop any self-check clause the user's original prompt carried over. Techniques 1 and 2 (investigate before answering, ground in quotes) still apply — they are about reading sources, not re-verifying output.

3. **Constrain scope on narrow tasks.** Opus 5 can add steps that were not requested or reinterpret what the task should be:

   ```text
   Deliver the scope that was asked. Make routine calls yourself; check in only when two readings of
   the request would produce materially different work. If the request looks mistaken, say so in one
   sentence and continue as asked, rather than narrowing, widening, or transforming it. Finish the whole
   task and stop there.
   ```

4. **Cap subagent delegation.** Opus 5 "delegates to subagents more readily than prior models" — this is the opposite of the Opus 4.8 tuning, where delegation had to be encouraged. Give explicit criteria or a deterministic cap:

   ```text
   Delegate only work that is large, genuinely independent, and parallelizable — a wide multi-file
   investigation, for instance. Do not delegate what you can finish in a few tool calls, and never
   delegate verification of your own work. One subagent beats several; keep the count low.
   ```

5. **Limit correction narration.** Opus 5 narrates corrections to its own earlier statements more than prior models, which reads as thrash in a user-facing product:

   ```text
   Correct an earlier statement only when the error would change the user's code, conclusions, or
   decisions: say it plainly and briefly, then continue. For slips that change nothing, fix them and
   move on without noting it.
   ```

6. **Describe the narration cadence you want.** Opus 5 announces what it is about to do and its per-message output in agentic sessions runs long. Tune it down by describing shape and cadence, not by prohibitions — "positive examples of the communication style you want tend to be more effective than instructions about what not to do":

   ```text
   One sentence before the first tool call, saying what you are about to do. While working, update only
   on a real finding or a change of direction. At the end, lead with the outcome: the first sentence
   answers "what happened" or "what did you find", the detail comes after it.
   ```

7. **Calibrate written deliverable length.** Separate from conversational verbosity: files Opus 5 writes to disk (reports, Markdown, summaries) run longer than on prior models. If the artifact is a document, add: `"Match the length of written documents to what the task needs: cover the substance, but do not pad with filler sections, redundant summaries, or boilerplate."`

8. **Keep thinking on; control cost with `effort` instead.** Thinking is on by default and can be disabled only at `effort` `high` or below. With thinking disabled two artifacts can appear: a **tool call written into the visible text** (the turn completes, the call never runs, and in an agentic loop the leaked text pollutes later turns), and **internal `<thinking>` tags in the response**. "For most tasks, thinking enabled at `low` effort performs better than thinking disabled at similar cost." If an integration must keep thinking off, use one combined instruction — and note the two counterintuitive rules: **remove** any rule telling the model not to think or not to reason (it increases tag leakage), and do **not** name thinking tags (the general form is more effective):

   ```text
   You may say one brief sentence before using a tool. If no tool can do what the user asked, say so
   rather than guessing. Never include internal or system XML tags in the response.
   ```

## Effort

Default is `high`. Use `low` and `medium` liberally as the primary control for token cost and response time wherever quality holds — on Opus 5 they "produce strong quality at a fraction of the tokens and latency of higher settings" — and step up to `xhigh` for demanding coding and agentic work. If effort defaults were carried over from a prior model, re-run an effort sweep on your own evals rather than assuming they transfer.

## Task-type notes

- **Long-horizon agentic work.** Opus 5 "performs best when given the complete task specification up front and left to run," and completes full tasks rather than leaving stubs or placeholders. Prefer one well-specified opening turn over building the spec up across interactive turns.
- **Code review.** High precision *and* recall. But "only report high-severity issues" / "be conservative" is followed literally and suppresses real findings — ask for everything with a confidence and severity label, then filter in a separate pass.
- **Vision.** Strong on charts, documents, diagrams, and UI replication. The high-leverage move is giving it tools to iteratively analyze, crop, and visually verify its work: "tool use is a more cost-effective lever than thinking alone." Re-validate prompt-side vision workarounds written for prior models — several are no longer needed.
- **Office and document tasks.** Handles multi-sheet spreadsheets with non-trivial formulas and well-structured decks; supply the specific style or template it must follow.
- **Long context.** 1M-token window as both default and maximum, with instruction following, tool calling, and reasoning consistent across the window.

## Refusals

Opus 5 ships with elevated cybersecurity safeguards and can return `stop_reason: "refusal"` on a successful HTTP response. For production pipelines, configure a [refusal fallback](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback) — the `fallbacks: "default"` mode lets Anthropic route by refusal category instead of pinning a substitute model. Check `stop_reason` before reading the response content.

## What this means for the 7 components

| Component | Effect on Opus 5 |
|---|---|
| Output specification | **Up.** Length and tone must be prompted; `effort` does not shorten visible output. |
| Constraints | **Up, but different.** Scope discipline and a delegation cap earn their place; verification and re-check rules are removed, not reworded. |
| Task | Unchanged and still load-bearing — give the complete spec up front for long-horizon work. |
| Examples | Still the first component to cut (see `framework.md` > "Calibrating for frontier models"); positive style examples are the exception worth keeping when tuning narration. |
| Role / Context / Structure | Unchanged. XML tags remain the strongest structure format on Claude. |
