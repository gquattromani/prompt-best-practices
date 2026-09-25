# Prompt Best Practices

Before executing an underspecified request, structure the prompt first. A vague goal produces an assumption-filled answer and several rounds of corrections; a prompt sized to the task does not.

## When to assess

Assess when the request is an **execution request** (write, draft, generate, implement, fix, refactor, build, create, design, analyze) that produces an artifact AND carries fewer components than its tier requires.

Do not assess: micro-tasks (rename one identifier, fix a typo, add a missing import, apply a linter suggestion), questions and conversation ("what is X?", "how does Y work?"), or when the user says "just do it", "execute now", "skip optimization".

## Task tiers

| Tier | Weight | Required components | Question cap |
|---|---|---|---|
| Quick | trivial, self-contained (1-3 min of human work) | 2 (Task + Output spec) | 2 |
| Standard | bounded, has stakes or conventions (15-60 min) | 3-4 | 3 |
| Complex | open-ended, multiple stakeholders or accuracy-critical (>1 hr) | 5+ | 5 |

## The 7 components

1. **Task** — clear action with measurable success criteria.
2. **Role** — the expertise the agent should bring.
3. **Context** — documents, files or data to read first.
4. **Examples** — one concrete example of the desired result beats three descriptions of it.
5. **Output specification** — format, length, tone, audience.
6. **Constraints** — rules, each stated once with its reason.
7. **Structure** — tagged sections (XML for Claude, Markdown or JSON elsewhere).

## Match the runtime

The components are the same everywhere; the dialect is not. Resolve the target before writing the prompt: a model or vendor named by the user wins; otherwise infer from the host — Claude Code → Anthropic Claude, Codex → OpenAI GPT, Gemini CLI / Antigravity / Jules → Google Gemini, Grok Build → xAI Grok; otherwise use the universal baseline. Never guess a vendor from a multi-model host (GitHub Copilot, Cursor, Windsurf, Cline, Zed, Kiro, Qoder, OpenCode, Aider, Amp, Junie, Devin, pi, Swival, OpenClaw, CodeWhale) — say in one line which runtime the prompt assumes so the user can correct it. Resolving the runtime is never one of the tier's questions.

| Target | Compose it this way |
|---|---|
| Anthropic Claude | XML tags. Long data at the top, task at the end. **Remove** self-check and verification clauses on Opus 5 / 5.5 (keep them on Fable long runs). State response length explicitly — `effort` does not shorten output. Reasoning depth is the `effort` parameter, never "think hard" in text. |
| OpenAI GPT | Markdown headings or XML. Lean: no repeated instruction, no example that does not change behavior. One authorization boundary, stated once — on GPT-6 it also grants the autonomy the request implies. State the writing style. The `apply_patch` tool when the task edits files. |
| Google Gemini | Markdown headings or XML tags, one or the other consistently. Data first, instruction last, anchored with "Based on the information above". State length and tone — the default is terse. Carry **no** sampling instruction: the vendor says to remove sampling parameters. Keep an example when it pins a format; drop chain-of-thought scaffolding. "Think very hard before answering" is legitimate here, on heavy-reasoning tasks only. |
| xAI Grok | The one runtime that asks for a *thorough* prompt: keep Role, spell out edge cases in the constraints, enumerate the context paths and exclude the rest. Never request tool calls in an XML envelope — native function calling instead. |
| Unknown, or a multi-model host | Universal baseline: XML tags (a hidden model may be Claude, and no other vendor penalizes them), context first, at most one example, explicit length, rules with their reason, and no vendor-specific parameter text (`temperature`, `effort`, `thinking_level`, `verbosity`), no self-check clause, no prefill. |

## Workflow

1. Classify the tier, then mark each of the 7 components `[OK]`, `[~~]` or `[--]`.
2. Present the diagnosis: tier, the marks with a one-line rationale each, and the exit — the user can reply `skip`, `go`, `execute` or `as-is` to run the original prompt unchanged. Always offer it.
3. Ask ONE question per message, in priority order (Task and success criteria, Output spec, Examples, Constraints, Role and Context), never more than the tier cap. Stop early if the user signals impatience.
4. Build the *smallest sufficient* prompt, not the most complete one. A component earns its place only if it changes the output.
5. Present the prompt, then ask for one-word confirmation before executing.
6. On a refinement request, adjust only the named components. Do not restart the dialogue.

## Rules

- Frontier models reward leaner prompts. Padding a quick-tier prompt up to seven components makes it worse, not better.
- Per-vendor and per-model guidance is not additive: remove self-check clauses on Claude Opus 5 / 5.5 and keep them on Claude Fable long runs; cap subagent delegation on Opus, request it on GPT-6; trim for Claude, GPT and Gemini but not for Grok, which asks for detail. Applying the wrong column is a regression, not a style choice.
- Set reasoning depth with the runtime's parameter (`effort`, `reasoning.effort`, `thinking_level`, `reasoning_effort`), not with "think hard" text — Google Gemini on heavy-reasoning tasks is the single documented exception.
- No emoji. Use the text markers `[OK]`, `[~~]`, `[--]`.
- Reply in the language of the user's prompt.
- Collaborative, never judgmental. One line of explanation per concept.

Full workflow, component rubrics, host-to-vendor routing and per-model tuning: `skills/prompt-best-practices/SKILL.md`, then `references/runtime-detection.md`.

<!-- shared-rule-end -->

## Repository map

This repository is an Agent Skill following the [Agent Skills open standard](https://agentskills.io/). The text above is the compact always-on rule for agents without skill support; everything below is for agents working *on* this repository.

| Skill | Location | Description |
|---|---|---|
| prompt-best-practices | `skills/prompt-best-practices/SKILL.md` | Interactive prompt structuring using a 7-component framework derived from Anthropic's published prompting guide |

Reference files in `skills/prompt-best-practices/references/` provide detailed topic guidance. Each one is self-contained, opens with a `## When to Use` section, and stays under 4000 tokens so it can be loaded in full. `SKILL.md` > Reference Map is the authoritative "load this when" table.

- `framework.md` — **entry point**: task tiers, the 7 components at a glance, the "Calibrating for frontier models" guidance, the final prompt template
- `component-definitions.md` — canonical definition of each of the 7 components (format, what to check for, present/missing criteria)
- `component-rubrics.md` — operational checklist for classifying each component as `[OK]` / `[~~]` / `[--]`
- `runtime-detection.md` — **top-level router**: maps the host the skill runs in to the model family that will consume the prompt, and holds the cross-vendor divergence table
- `universal-baseline.md` — the vendor-agnostic intersection, used when the runtime cannot be resolved
- `examples-content.md` / `examples-code.md` — dialogue examples (content tasks / code tasks)
- `grounding-techniques.md` — techniques to prevent hallucinations (technique 3, self-check, is model-gated)
- `claude-considerations.md` — Anthropic Claude router: per-model file selection, family-wide behaviors, and the divergence table where per-model tuning conflicts
- `claude-opus-5-5.md` — tuning notes for Claude Opus 5.5 (default target) and Claude Opus 5
- `claude-fable-5.md` — tuning notes for Claude Fable 5.1 / Mythos 5.1 and Fable 5 / Mythos 5 (highest-capability tier)
- `codex-considerations.md` — tuning notes for OpenAI Codex (GPT-6 Sol / Astra / Luna; GPT-5.6 previous generation)
- `gemini-considerations.md` — tuning notes for Google Gemini (Gemini 3.x, 3.8 Flash current: terse defaults, no sampling parameters, `thinking_level`)
- `grok-considerations.md` — tuning notes for xAI Grok (`grok-4.7`: thorough prompts, native tool calling)
- `maintenance.md` — **contributor-only**: verification record, source fingerprint, update procedure. Do not load it to build a prompt.

Test assets in `tests/`:

- `activation-fixtures.md` — reference prompts with expected activation outcomes. Run before merging any change to `SKILL.md`, `framework.md`, `component-definitions.md`, or `component-rubrics.md`. Fixture set F is three shell commands that verify token budgets, the `## When to Use` headers, and that every path named in `SKILL.md` resolves. Fixture set G verifies the per-platform adapters; fixture set H verifies host-to-vendor routing.
- `benchmark-protocol.md` — protocol for measuring the framework's effect on output quality.

Host adapters (which file each platform reads) are documented in `docs/agent-portability.md`. The rule text above the `shared-rule-end` marker is canonical: every copy under `.cursor/`, `.windsurf/`, `.clinerules/`, `rules/`, `.qoder/`, `.kiro/`, and `.github/copilot-instructions.md` must match it (fixture G1).
