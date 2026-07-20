# Component Rubrics

## When to Use

Load this file whenever you need to classify whether a component is `[OK]`, `[~~]`, or `[--]` in a prompt. The rubrics below turn subjective judgment into a concrete checklist, so two runs on the same prompt produce the same diagnostic.

The checklist in this file is authoritative for the `[OK]/[~~]/[--]` marks shown in `SKILL.md` > Step 1. `framework.md` remains the canonical source for what each component *is*; this file defines *how to count it*.

## Classification key

- `[OK]` — all required checks pass.
- `[~~]` — at least one required check passes, at least one fails.
- `[--]` — no required check passes.

Always cite which checks passed and which failed when presenting the diagnostic — this makes the classification auditable.

---

## Rubric 1 — Task

**Required checks:**

- **T1. Action verb.** The prompt contains an imperative verb targeting artifact creation (write, build, create, generate, draft, implement, design, refactor, fix, analyze, migrate). Not "tell me about" or "explain".
- **T2. Success criterion.** At least one of: a measurable metric (e.g., "under 200 words", "100% coverage", "under 200ms"), an observable behavior (e.g., "user books a call", "tests pass"), or a time-bounded outcome (e.g., "reply within 48h"). "Useful" / "good" / "clean" alone does not count.

**Classification:**

- `[OK]` — T1 and T2 both pass.
- `[~~]` — T1 passes, T2 fails. (Action is clear but success is undefined.)
- `[--]` — T1 fails.

---

## Rubric 2 — Role

**Required checks:**

- **R1. Named persona or expertise.** The prompt names a role ("senior backend engineer", "brand copywriter", "financial analyst"), an expertise area ("specializing in X"), or an implied-but-unambiguous capability (e.g., "you are an editor").

**Classification:**

- `[OK]` — R1 passes with a specific role (domain named).
- `[~~]` — R1 passes but generic ("you are a helpful assistant", "an expert").
- `[--]` — R1 fails.

**Notes:** Role is optional by default. For tasks where the model's generalist behavior is adequate, `[--]` is not a gap — it only matters for specialized tasks (security audits, legal review, brand copy, medical summaries).

---

## Rubric 3 — Context

**Required checks:**

- **C1. Context present.** The prompt references or includes background material: a document, a file path, a codebase area, a data sample, an audience profile, a style guide.
- **C2. Context structured.** The context is either (a) placed at the top of the prompt before the task, and (b) wrapped in a delimiter (XML tag, Markdown heading, fenced block) that separates it from the instructions.

**Classification:**

- `[OK]` — C1 and C2 both pass.
- `[~~]` — C1 passes, C2 fails. (Context is there, but mixed into instructions.)
- `[--]` — C1 fails.

**Notes:** For self-contained tasks (generate a haiku, draft a commit message), `[--]` is expected — context is not needed.

---

## Rubric 4 — Examples

**Required checks:**

- **E1. Concrete example present.** At least one example of the desired output — input/output pair, reference snippet, "like X's style", or a sample of the target format.
- **E2. Example is structured.** Wrapped in a delimiter (`<example>`, Markdown heading, fenced block) so the model distinguishes it from instructions.

**Classification:**

- `[OK]` — E1 and E2 pass, and there is at least one example.
- `[~~]` — E1 passes (a reference is mentioned) but not structured, or only abstractly described ("make it like the others").
- `[--]` — E1 fails.

**Notes:** The Anthropic guide recommends 3-5 examples for best results, but a single well-placed example is still `[OK]` for the purpose of this rubric. Count ≥3 diverse examples as a plus, not a requirement.

---

## Rubric 5 — Output specification

Sub-fields (from `framework.md` > Component 5):

- **O1. Format** — output type and length (e.g., "markdown, 200-400 words").
- **O2. Tone** — described positively ("warm, direct, advisor-like"), not only via anti-patterns.
- **O3. Audience impact** — what effect the output should produce on the reader.
- **O4. Success metric** — measurable outcome, aligned with Task's T2.
- **O5. Anti-patterns (optional)** — phrasings or tones to avoid. Secondary to O2.

**Classification:**

- `[OK]` — at least 3 of {O1, O2, O3, O4} pass. (O5 does not count toward `[OK]`.)
- `[~~]` — 1-2 of {O1, O2, O3, O4} pass.
- `[--]` — none pass.

**Notes:** O5 without O2 means the user defined what to avoid without defining what to produce. That is `[~~]` at best. See `framework.md` > Component 5 > "On the coexistence of Tone and Avoid".

---

## Rubric 6 — Constraints

**Required checks:**

- **K1. Explicit rule(s).** At least one rule stated as a constraint ("never use ellipses", "paragraphs under 4 lines", "use only metric units").
- **K2. Motivation.** Each rule includes a short reason ("because the output is read by a TTS engine", "mobile reading", "European audience").

**Classification:**

- `[OK]` — K1 passes with ≥1 rule, K2 passes for every rule (each rule has a reason).
- `[~~]` — K1 passes, but one or more rules lack a motivation.
- `[--]` — K1 fails.

**Notes:** Motivation is not a formality — the Anthropic guide explicitly says models generalize better from the explanation than from the bare rule. A long list of reason-less rules is `[~~]`, not `[OK]`.

---

## Rubric 7 — Structure

**Required checks:**

- **S1. Delimiters used.** At least 2 distinct sections are wrapped in consistent delimiters (XML tags, Markdown headings, fenced blocks, JSON keys).
- **S2. Consistency.** Delimiter style is uniform across the prompt (don't mix XML and Markdown headings for the same kind of content).

**Classification:**

- `[OK]` — S1 and S2 both pass.
- `[~~]` — S1 passes, S2 fails. (Delimiters used but inconsistent.)
- `[--]` — S1 fails.

**Notes:** Structure is format-agnostic (see `framework.md` > Component 7). XML is the Claude default but is not required for `[OK]` — consistent Markdown headings qualify.

---

## Worked example

Prompt: *"Write a follow-up email for a client who hasn't responded in 2 weeks. Keep it under 6 lines. The client should reply confirming they want to proceed."*

| Component | Checks | Result |
|---|---|---|
| Task | T1 pass ("write"), T2 pass ("client replies confirming") | `[OK]` |
| Role | R1 fail (no role) | `[--]` |
| Context | C1 fail (no background material) | `[--]` |
| Examples | E1 fail (no reference) | `[--]` |
| Output spec | O1 pass ("under 6 lines"), O2 fail, O3 fail, O4 pass (reply confirming) → 2/4 | `[~~]` |
| Constraints | K1 fail | `[--]` |
| Structure | S1 fail (flat text) | `[--]` |

Tier: quick (2-required). Already `[OK]` on Task; `[~~]` on Output spec. One quick question on tone completes the quick tier → execute.

---

## Update procedure

When a component definition changes in `framework.md`, update the corresponding rubric in this file before shipping. The rubric and the definition are tightly coupled — drift between them causes unreproducible diagnostics.
