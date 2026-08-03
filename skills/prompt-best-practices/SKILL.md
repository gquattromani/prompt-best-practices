---
name: prompt-best-practices
description: Optimize unstructured prompts into high-quality structured prompts. Triggers on execution requests (write, draft, generate, implement, fix, refactor, build) that lack clear success criteria or output shape. Skips micro-tasks (single-identifier renames, typo fixes, single-import additions) and conversational questions. Uses an adaptive 3-tier workflow — quick / standard / complex — so the dialogue length matches the task weight. Invoked via slash command it always assesses, but fast-tracks trivial prompts after one confirmation.
---

# Prompt Best Practices Skill

> **MANDATORY on slash-command invocation**: proceed to Step 0 — Quick Assessment. Do not refuse. The user can shortcut execution at Step 1 (see Fast-Track Exit).

## Purpose

Intercept unstructured prompts and guide users through an adaptive dialogue that builds a prompt sized to the task. The skill uses a modular 7-component framework derived from [Anthropic's official prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices).

## Reference Map

Reference files live in `references/`, relative to this file. Each one is self-contained, opens with its own `## When to Use`, and stays under 4000 tokens so it can always be loaded in full. Load a file when its row applies, and read it rather than answering from memory of it.

| Load this | When |
|---|---|
| `references/framework.md` | **Start here.** Task tiers, the 7 components at a glance, frontier-model calibration, the final prompt template. Used by Step 0 and Step 3. |
| `references/component-definitions.md` | A component's formal definition is in question — format, what to check for, what counts as present. |
| `references/component-rubrics.md` | Assigning `[OK]` / `[~~]` / `[--]` in Step 0 Pass B. Authoritative for the marks. |
| `references/claude-considerations.md` | The target runtime is Claude. It is the router: it names the per-model file to load and holds the divergence table. |
| `references/claude-opus-5.md` | The target is Claude Opus 5 — assume this when the user states no model. |
| `references/claude-fable-5.md` | The target is Claude Fable 5 / Mythos 5. |
| `references/codex-considerations.md` | The target is OpenAI Codex (GPT-5.6 family). |
| `references/grounding-techniques.md` | The task is accuracy-critical (Step 2, priority 4). Technique 3 is model-gated. |
| `references/examples-code.md` / `references/examples-content.md` | A worked end-to-end dialogue helps — code tasks / content tasks respectively. |

`references/maintenance.md` is contributor-only (verification record, update procedure). Never load it to build a prompt.

**If a reference file cannot be read**, continue with this file alone and say so in one line. The workflow below is self-sufficient for tier detection, the dialogue, and the final template — an unavailable reference is not a reason to skip the workflow.

## Non-goals

- Not a generic writing coach. If the user asks a question, wants conversation, or describes a micro-task, do not activate.
- Not an empirical guarantee. The framework encodes well-documented heuristics. See `tests/benchmark-protocol.md` for how contributors measure effect size.
- Not XML-only. Structure is format-agnostic. XML is the default for Claude; Markdown or JSON is valid elsewhere (see `references/component-definitions.md` > Component 7).

## Activation Rules

### ALWAYS ASSESS when:
- The user explicitly invokes this skill via `/prompt-best-practices`. Assessment is mandatory; the dialogue itself is shortcut-able via the Fast-Track Exit in Step 1.

### AUTO-ACTIVATE when all of the following are true:
1. The request is an **execution request** (verb: write, draft, generate, implement, fix, refactor, build, create, design, analyze) that produces an artifact.
2. The task is **standard or complex** tier (see `references/framework.md` > Task Tiers). Quick-tier tasks are not auto-activated — they are self-contained enough that a 2-component prompt is sufficient.
3. The prompt has **fewer components than the tier requires** (quick: 2, standard: 3-4, complex: 5+ — see `references/framework.md`).

### DO NOT ACTIVATE when:
- The request is a **micro-task** (rename a single identifier, fix a typo, add a missing import, apply a linter suggestion). The codebase supplies all the context the model needs.
- The request is a **question** or conversational exchange ("what is X?", "how does Y work?", "explain Z").
- The user explicitly asks to execute immediately ("just do it", "execute now", "skip optimization", "no prompt").
- The prompt meets the tier's component threshold (checked via `references/component-rubrics.md`).

### Fast-Track Exit (all activation modes)
At Step 1 the user can type `skip`, `go`, `execute`, or `as-is` (any language equivalent) to exit the dialogue and run the original prompt unchanged. This is the documented escape; always offer it in the Step 1 message.

## Workflow

### Step 0 — Quick Assessment

Two-pass assessment:

**Pass A: Task tier classification.** Read the prompt and pick one tier (definitions in `references/framework.md` > Task Tiers):
- **Quick** — trivial, self-contained artifact (1-3 minutes of human work). Example: "write a commit message for this diff". Required components: 2 (Task + Output spec).
- **Standard** — bounded artifact with stakes or conventions (15-60 min of human work). Example: "write a PR description" or "add a Zod schema validator". Required components: 3-4 (Task + Output spec + Examples or Constraints).
- **Complex** — open-ended artifact with multiple stakeholders, accuracy-critical constraints, or external context (>1 hr of human work). Example: "migrate the auth middleware to JWT". Required components: 5+ (all that add value).

**Pass B: Component count.** Check the 7 components against `references/component-rubrics.md` and mark each `[OK]`, `[~~]`, or `[--]`. The rubric turns subjective judgment into a checklist; `references/framework.md` > "The 7 components at a glance" is the short list of what is being counted, and `references/component-definitions.md` has the full definition when a call is genuinely ambiguous.

Slash-command invocation always proceeds to Step 1. Auto-activation follows the Activation Rules.

### Step 1 — Diagnosis Message

Present the diagnostic with the tier, the component grid, and the Fast-Track Exit offer.

Template:
```
Prompt analysis:

Tier detected: {quick|standard|complex} (required: {N} components)

[OK] Task          — <one-line rationale>
[~~] Role          — <one-line rationale>
[--] Context       — <one-line rationale>
[--] Examples      — <one-line rationale>
[--] Output spec   — <one-line rationale>
[--] Constraints   — <one-line rationale>
[--] Structure     — <one-line rationale>

{M} of {N} required components are already in place.

Want to build the missing pieces together (≈{K} quick questions), or should I run it as-is?
Reply: "go" to build, "skip" to execute unchanged.
```

`K` is the remaining dialogue length (never more than the tier's ceiling: quick=2, standard=3, complex=5).

### Step 2 — Guided Dialogue (Adaptive)

Ask ONE question per message, capped by tier:
- **Quick** tier: max 2 questions (Task success criteria, then Output spec).
- **Standard** tier: max 3 questions (Task, Output spec, + one of {Examples, Constraints}).
- **Complex** tier: max 5 questions (full priority order below).

Priority order (run only the top `K` from this list, where K = tier cap minus components already `[OK]`):

1. **Task + Success Criteria** — "What outcome do you need, and how will you know it succeeded?"
2. **Output specification** — "What should the output look like (format, length, tone)?" Lead positive; ask anti-patterns only if the user volunteers them or the task is accuracy-critical.
3. **Examples** — "Do you have a reference example of the desired result? Even one helps."
4. **Constraints** — "Any rules to respect (style guide, architecture, brand voice)? For each, the reason matters." For accuracy-critical tasks, suggest grounding constraints from `references/grounding-techniques.md` — but skip the self-check constraint when the target runtime is Claude Opus 5, where it causes over-verification.
5. **Role + Context** — "Is there a specific expertise the agent should bring, and any files/documents to read first?"

Dialogue rules:
- Keep each question's explanation to 1 line max.
- Give 2-3 concrete example answers tailored to the user's prompt.
- If the user says "I don't know", suggest a reasonable default and confirm in one sentence.
- If the user signals impatience ("let's go", "that's enough", "hurry"), stop asking and build the prompt with what you have.
- Obey the tier cap. Do not ask more questions than the tier allows, even if components are still missing — the framework is modular.

### Step 3 — Build Final Prompt

Construct the final prompt using the template in `references/framework.md` > Final Prompt Template. Apply Component 7 (Structure) using the format appropriate for the target runtime:
- **Claude** (Opus 5 default target; Fable 5 / Mythos 5 top tier; Opus 4.8 previous generation) → XML tags (default, strongest). Start at `references/claude-considerations.md`, then load the per-model file.
- **Codex / OpenAI** (GPT-5.6 Sol flagship) → Markdown headings or XML (both work). See `references/codex-considerations.md`.

Adapt the template: include only the sections the tier and dialogue produced. Quick-tier prompts with just `<task>` and `<output_spec>` are valid and preferred to padding.

**Frontier-model calibration (Opus 5, Fable 5 / Mythos 5, GPT-5.6):** build the *smallest sufficient* prompt, not the most complete one. All three explicitly reward leaner prompts and can degrade on over-prescription. A component earns its place only if it changes the output — examples are the first to cut, rules are stated once with their reason, and reasoning depth is set by the `effort` parameter rather than "think hard" text. Some legacy instructions are a *delete*, not a rewrite. See `references/framework.md` > "Calibrating for frontier models".

**Two model-gated checks before presenting the prompt** (both from `references/claude-considerations.md` > "Pick the model before you tune"):
- If the target is **Claude Opus 5**, remove any self-check / verification / "double-check your answer" clause — including one the user's original prompt carried over — and note the removal in one line. Response length must be stated in `<output_spec>`; `effort` will not shorten it.
- If the target is **Claude Fable 5 / Mythos 5** and the task is a long autonomous run, the verifier instruction is the opposite call: keep it, preferring a fresh-context verifier subagent over self-critique.

Present the prompt, then ask for one-word confirmation before executing.

### Step 4 — Refinement (optional)

If the user asks to modify the generated prompt, adjust only the requested components and re-present. Do not restart the dialogue.

## Tone and Style

- Collaborative, never judgmental.
- Brief explanations (1 line per concept).
- Do not use emoji — use text indicators `[OK]`, `[--]`, `[~~]`.
- Respect the tier cap: do not extend the dialogue past the ceiling.
- Respond in the same language as the user's prompt. If ambiguous, ask.

## Edge Cases

- **Partial answers**: incorporate what the user gives, suggest defaults for gaps, confirm in one sentence, move on.
- **Components supplied upfront in a single message**: skip what is already there; ask only about what the tier still requires.
- **Mixed intent (question + execution request)**: answer the question directly, then offer optimization only for the execution part.
- **Target runtime is Claude Opus 5 (the default when the model is unstated)**: consult `references/claude-opus-5.md` — prompt explicitly for conciseness (`effort` does not shorten visible output), delete verification and self-check clauses instead of rewording them, constrain scope on narrow tasks, cap subagent delegation, and keep thinking on rather than disabling it. The 7 components still apply: Output specification and Constraints carry more weight, verification-style constraints are dropped.
- **Target runtime is OpenAI Codex (GPT-5.6 family)**: consult `references/codex-considerations.md` — outcome-first / leaner prompts (remove repeated instructions and non-behavioral examples), `apply_patch` edit format, autonomy-and-approval boundaries stated once, `reasoning.effort` for depth, multi-agent / programmatic tool calling. The 7 components still apply, but through the lean-prompt lens.
- **User invoked the skill by mistake**: Fast-Track Exit is the documented escape. Do not argue; run the original prompt.
