# Runtime Detection and Vendor Routing

## When to Use

Load this file **before any vendor file**, at the start of Step 3 (Build Final Prompt) and whenever a dialogue answer depends on the target runtime. It answers one question: which model family will consume the prompt, and therefore which dialect to write it in. It is the top-level router — `claude-considerations.md` is the second-level router for the Anthropic branch.

The 7 components never change. What changes is the structural format, whether examples earn their place, how verbosity and reasoning depth are controlled, and which clauses must be deleted rather than reworded.

## The routing rule

Three steps, in order. Stop at the first that resolves.

1. **An explicit statement wins.** If the user names a model, a vendor, or an API ("this goes to Gemini 3.1 Pro", "for our GPT-5.6 pipeline"), route on that and ignore the host. The prompt may be written in one agent and executed in another; the destination is what matters, not where the dialogue happens.
2. **Otherwise infer from the host.** Use the map below. Single-vendor hosts resolve with no further work.
3. **Otherwise use `universal-baseline.md`.** A multi-model host with no visible selection is not a reason to guess: the baseline is the vendor-agnostic intersection and is a correct answer, not a degraded one.

**Do not spend a tier-capped dialogue question on the runtime.** It is a routing decision, not a component — the caps in `SKILL.md` > Step 2 exist for the 7 components. When the target is genuinely load-bearing and unresolved (complex tier, or the prompt will be reused across runtimes), state the assumption in one line while presenting the prompt in Step 3, phrased so the user can correct it without a round trip: "Written for Claude Opus 5 (XML tags, no self-check clause) — say the word if it is going somewhere else."

**Never infer a vendor from a multi-model host.** Writing XML-heavy Claude structure for a Gemini-backed session, or carrying a `temperature: 0.2` instruction into a Gemini 3 prompt whose vendor says keep the default at 1.0, is a measurable regression, not a stylistic difference.

## Host → vendor map (verified 2026-08-19)

| Host | Model family | Load | Kind |
|---|---|---|---|
| Claude Code (CLI, desktop Code tab, IDE extensions) | Anthropic Claude | `claude-considerations.md`, then the per-model file | Single-vendor |
| Codex (CLI, desktop app, VS Code Codex extension) | OpenAI GPT | `codex-considerations.md` | Single-vendor |
| Gemini CLI | Google Gemini | `gemini-considerations.md` | Single-vendor |
| Jules | Google Gemini | `gemini-considerations.md` | Single-vendor |
| Antigravity CLI (`agy`) / Antigravity IDE | Google Gemini by default; Claude and open models are selectable in the picker | `gemini-considerations.md` unless the picker says otherwise | Default known, switchable |
| Grok Build | xAI Grok (`grok-4.6`) | `grok-considerations.md` | Single-vendor |
| GitHub Copilot (CLI, editor extensions, cloud agent) | Vendor picker: Anthropic, OpenAI, Google, xAI, and others | The vendor file for the selected model, else `universal-baseline.md` | Host-selected |
| Cursor, Windsurf, Cline, Zed, Kiro, Qoder, OpenCode, Aider, Amp, Junie, Devin, pi, Swival, OpenClaw, CodeWhale | Host- or user-selected, frequently changed | The vendor file if the selection is visible, else `universal-baseline.md` | Host-selected |
| API integration, CI job, unknown harness | Unknown | `universal-baseline.md` | — |

**Reading the selection on a host-selected runtime**, cheapest first: the model named in the session UI or statusline; the model the user mentioned earlier in the conversation; a config file already in context (`~/.grok/config.toml`, a `model` key in the host's settings, a note in `AGENTS.md`). If none of those is already available, stop — do not spend a tool call to discover it on a quick- or standard-tier prompt. Route to `universal-baseline.md` instead.

This map is the most perishable content in the skill: hosts add vendors, switch defaults, and get renamed. Treat a row as stale if the host's own documentation disagrees with it, and follow the update procedure below.

## Cross-vendor divergence

Where the vendors actively disagree. Applying the wrong column is not neutral — each cell below is something a vendor states about its own models.

| Behavior | Anthropic Claude | OpenAI GPT-5.6 | Google Gemini 3.x | xAI Grok 4.6 |
|---|---|---|---|---|
| Structure format | XML tags are the strongest tool | Markdown headings or XML, no preference | Clear delimiters — XML-style tags **or** Markdown headings | Not specified for sections; **native tool calling instead of XML tool-call output** |
| Prompt size | Lean; delete legacy scaffolding | Lean; leaner system prompts scored ~10-15% higher with 41-66% fewer tokens ([source](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6)) | "Be concise in your input prompts" | **Detailed system prompt recommended** — the one vendor that asks for more, not less |
| Examples | First component to cut | Only when one example changes behavior | Recommended to regulate format; simplify prompts that only needed them for chain-of-thought | Concrete, specific context over generic examples |
| Reasoning depth | `effort` parameter; never "think hard" in text | `reasoning.effort`; never in text | `thinking_level`; **and** "Think very hard before answering" can help on heavy-reasoning problems | `reasoning_effort` (`low`…`xhigh`), cannot be disabled |
| Verbosity | Must be prompted explicitly — `effort` does not shorten visible output | `text.verbosity` plus a length line in the prompt | Terse by default; ask explicitly for a conversational or longer answer | Not documented |
| Self-check clause | **Remove** on Opus 5; keep on Fable 5 long runs | State approval boundaries once, no per-action nagging | Not documented; use Search grounding and code execution instead | Not documented |
| Sampling parameters | Not a prompt concern | Not a prompt concern | **Keep temperature at the 1.0 default**; lowering can cause looping | Not documented |
| Context placement | Long data at the top, query at the end — the vendor states it [improves performance across all models](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting) | Domain context near the top | Data first, instructions and questions last, with an anchor phrase | Name the specific files, paths, and dependencies; exclude irrelevant context |
| Caching | Not a prompt-text concern | Explicit prompt-prefix caching | Not a prompt-text concern | `prompt_cache_key` strongly recommended for repeated prefixes |

Two rows deserve emphasis because they invert the usual advice:

- **xAI asks for detail where Anthropic, OpenAI, and Google ask for brevity.** A prompt trimmed for Gemini 3 can underperform on Grok. Do not carry the lean-prompt reflex across that boundary without checking `grok-considerations.md`.
- **Google endorses a reasoning nudge in prompt text** ("Think very hard before answering") that Anthropic and OpenAI explicitly replace with a parameter. This is the clearest case where copying a Claude prompt to Gemini leaves quality on the table, and vice versa.

## What never goes into a prompt whose vendor is unknown

- Vendor-specific parameter names or values as prose (`effort: high`, `thinking_level: low`, `text.verbosity`, `temperature: 0.2`). They are API fields, not instructions, and one vendor's safe value is another's regression.
- Assistant prefill, or any assumption that the last assistant turn can be seeded — unsupported on current Claude models.
- A tool-call output format ("reply with `<tool_call>` XML"), which conflicts with native tool calling on xAI and with structured tool APIs generally.
- "Reproduce your reasoning in the response", a refusal trigger on Claude Fable 5 and pointless elsewhere.
- Any claim about a context window, cutoff date, or model name. If the prompt names the model, it stops being portable.

## Update procedure

1. **Re-verify the host map first** — it rots faster than the vendor files. For every `Single-vendor` row, confirm the host has not added a picker; for every `Host-selected` row, confirm the host still exists under that name.
2. When a vendor ships a new generation, update its own file (`claude-considerations.md` and its per-model files, `codex-considerations.md`, `gemini-considerations.md`, `grok-considerations.md`) and then **re-diff the divergence table** above. New generations tend to invert prior guidance rather than extend it.
3. Add a vendor file only when a host in the map routes to it. A file no host reaches is dead content; a host with no file is a gap that silently falls back to the baseline.
4. Update the verification record in `maintenance.md` with the date and the pages checked, and re-run fixture set H in `tests/activation-fixtures.md`.
