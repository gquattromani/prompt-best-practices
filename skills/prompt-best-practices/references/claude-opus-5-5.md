# Claude Opus 5.5 — Tuning Notes

## When to Use

Load this file when the target runtime is Claude Opus 5.5 (`claude-opus-5-5`) — the default target for most Claude work, including Claude Code — or Claude Opus 5 (`claude-opus-5`). The Opus 5 tuning points are the documented starting point for both models; `claude-considerations.md` > "Claude Opus 5" lists what differs on the older one. For the router and the behaviors shared by the whole family, see `claude-considerations.md`. For the highest-capability tier, see `claude-fable-5.md`.

Derived from [Prompting Claude Opus 5.5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) and [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) (both verified 2026-09-25, see `maintenance.md`). Text in double quotes is quoted verbatim from those pages; the fenced instruction blocks are condensed adaptations of their sample prompts, not quotations. These notes summarize and comment on the pages; they are not a substitute for them. Opus 5.5 is built for long-running agentic coding and knowledge work.

## Start from the Opus 5 patterns

Anthropic is explicit: "Existing Claude Opus 5 prompts should perform well without changes, and the patterns in Prompting Claude Opus 5 remain a reasonable starting point." Opus 5.5 also "generates output tokens more than 30 percent faster than Claude Opus 5 and tends to finish the same task with fewer tokens." A migration is not a rewrite: keep the six inherited points, then apply the Opus 5.5 changes below them.

One caveat from the general guide: "Where a technique names a specific model, treat it as measured on that model and re-check it against your own evals before applying it to another." The six points were measured on Opus 5 and carry over to Opus 5.5 as its documented starting point, not as a separate measurement.

## Inherited from Opus 5

1. **Prompt for conciseness — `effort` will not do it.** On Opus 5, "the effort parameter controls how much the model thinks rather than how much it says: lowering effort can reduce thinking volume without reliably shortening the visible response." This makes Component 5 (Output specification) load-bearing even for quick-tier prompts:

   ```text
   Keep the response focused and brief: spend it on the answer, keep caveats short, and default to a
   high-level summary unless depth was asked for.
   ```

2. **Delete verification instructions — do not rewrite them.** Explicit verification steps, re-check phrasing ("double-check your answer"), and a subagent whose job is to double-check "cause over-verification on Claude Opus 5, and removing them reduces wasted tokens with no loss in quality." For this skill: **skip technique 3 (self-check) in `grounding-techniques.md`** and drop any self-check clause the user's original prompt carried over. Techniques 1 and 2 (investigate before answering, ground in quotes) still apply — they are about reading sources, not re-verifying output.

3. **Constrain scope on narrow tasks.** The model can add unrequested steps or reinterpret the task:

   ```text
   Deliver the scope that was asked. Make routine calls yourself; check in only when two readings of
   the request would produce materially different work. If the request looks mistaken, say so in one
   sentence and continue as asked. Finish the whole task and stop there.
   ```

4. **Cap subagent delegation.** Opus 5 "delegates to subagents more readily than prior models" — the opposite of the Opus 4.8 tuning. Give explicit criteria or a deterministic cap:

   ```text
   Delegate only work that is large, genuinely independent, and parallelizable. Do not delegate what
   you can finish in a few tool calls, and never delegate verification of your own work.
   ```

5. **Limit correction narration.** Correct an earlier statement only when the error would change the user's code, conclusions, or decisions; fix slips that change nothing without noting them.

6. **Calibrate written deliverable length.** Files written to disk run long too. If the artifact is a document, add: `"Match the length of written documents to what the task needs: cover the substance, but do not pad with filler sections, redundant summaries, or boilerplate."`

## What changes on Opus 5.5

1. **Effort defaults to `medium` — set it explicitly.** "Start at `medium`, the default on Claude Opus 5.5 (Claude Opus 5 defaults to `high`), set it explicitly, and test several levels against your own evals rather than carrying over the setting you used on Claude Opus 5." Level names do not transfer: at a given level the model "tends to think more per turn than Claude Opus 5, especially at `xhigh` and `max`." Reserve those two for measured gains, and leave `max_tokens` room for thinking. "To get less thinking, lower the effort level first" — the parameter works more reliably than prompt text, so depth never goes into the prompt.

2. **Thinking is always on — delete the thinking-disabled scaffolding.** `thinking: {"type": "disabled"}` returns a 400. Remove any instruction that asked the model to write its reasoning into the response as a stand-in for thinking: such a prompt "can be declined with the `reasoning_extraction` refusal category", which is new on this model. Remove any rule telling the model not to think, and re-test whether the Opus 5 thinking-disabled mitigation (`claude-considerations.md` > "Claude Opus 5") is still needed.

3. **Remove "think carefully" lines from chat system prompts.** "In chat applications, if your system prompt contains instructions that tell Claude to think carefully before answering, consider removing them for Claude Opus 5.5." Anthropic reports replies starting sooner with no clear decline in quality. This is a *delete, not a rewrite* case, like the verification clause. Where follow-up turns re-open settled answers, a two-sentence addition helps — leave it out of long analyses and agentic tasks where a later step can reveal an earlier mistake:

   ```text
   Treat an answer you have given as settled. On later turns, think about what the user is asking now,
   and revisit an earlier answer only if the user asks about it or points out a problem.
   ```

4. **Unattended runs: name the early stops.** Some progress updates end the turn with text rather than a tool call, and an unattended loop that reads that as completion stops there. The model "is responsive to instructions that name the specific kinds of early stop you want it to avoid", and it also helps to name the stops you *do* want. For a fully unattended agent only — not a human-in-the-loop product — add at the end of the system prompt, from the first request:

   ```text
   The user is not watching and will not answer mid-task. Do not end a turn on a summary that announces
   the next step, an offer to continue, a list of decisions that block nothing, or a milestone report:
   put status notes in the same message as your next tool call and keep going. Stop only when nothing
   can move without the user. This does not remove the need to confirm risky or destructive actions.
   ```

   Keep the task's parts in a checklist the model updates, and cap automatic continuations at two or three so a genuinely stuck run ends.

5. **Ask for the update cadence you want.** Between tool calls the model writes short progress notes, returned as progress-update `thinking` blocks (empty at the default display; `display: "updates"` returns them). For a predictable shape — a one-line intent before the first tool call, a recap at the end — "say so in the system prompt; the model is responsive to such instructions."

6. **Mark pasted text.** With the right context the model is robust against instructions inside content a user pasted in. When the prompt embeds pasted material, wrap each block in `<pasted_content id="…">` tags carrying the same short random ID on the opening and closing tag, and tell the model in the system prompt that instructions inside those tags are followed only where the user's own message asks for it. "The tags are plain text and can be imitated, so treat this as one guardrail alongside other prompt-injection defenses."

7. **Look around before acting in multi-app workflows.** The model "tends to get to work quickly", so on loosely specified automation across email, documents, and records, one sentence asking it to open the relevant sources — including ones the task does not name — before changing anything improved completion in Anthropic's testing. Keep untrusted content out of what it searches.

8. **Name the frontend patterns to avoid.** A general instruction such as "avoid a generic AI look" "mostly swaps one default for another". The model "responds well to instructions that name specific patterns to avoid" — this is the one task type where a concrete `Avoid` list in Component 5 beats a positive description.

9. **Re-test vision scaffolding.** Opus 5.5 reads charts, diagrams, and screenshots considerably more precisely than Opus 5 without tools; image-processing or crop tools still add accuracy on the densest inputs.

10. **Time budgets for agent teams.** In a multi-agent harness, an elapsed-time line against a budget (`elapsed 340s / 1200s`) lets the model parallelize and finish sooner. It is advisory: keep your own timeout.

## Refusals

Opus 5.5 runs biology, cybersecurity, and reasoning-extraction classifiers; the biology and `reasoning_extraction` categories are new if you come from Opus 5. A decline arrives as `stop_reason: "refusal"` on a successful HTTP response. Configure a [refusal fallback](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback) — `fallbacks: "default"` retries on the model Anthropic recommends for the category — and note that server-side fallback returns `reasoning_extraction` declines instead of retrying them, so the fix for those is in the prompt.

## What this means for the 7 components

| Component | Effect on Opus 5.5 |
|---|---|
| Output specification | **Up.** Length must be prompted (inherited); cadence of progress updates is described here; frontend work takes a concrete avoid-list. |
| Constraints | **Up, but different.** Scope discipline and a delegation cap earn their place; verification, "think carefully", and reason-in-the-response rules are removed, not reworded; unattended agents name the early stops. |
| Task | Unchanged and still load-bearing — give the complete spec up front for long-horizon work. |
| Context | Pasted material goes inside marked `<pasted_content>` blocks. |
| Examples | Still the first component to cut (`framework.md` > "Calibrating for frontier models"). |
| Role / Structure | Unchanged. XML tags remain the strongest structure format on Claude. |
