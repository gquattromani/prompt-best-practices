# Runtime Detection and Vendor Routing

## When to Use

Load this file **before any vendor file**, at the start of Step 3 (Build Final Prompt) and whenever a dialogue answer depends on the target runtime. It answers one question: which model family will consume the prompt, and therefore which dialect to write it in. It is the top-level router — `claude-considerations.md` is the second-level router for the Anthropic branch.

The 7 components never change. What changes is the structural format, whether examples earn their place, how verbosity and reasoning depth are controlled, and which clauses must be deleted rather than reworded.

## The routing rule

Three steps, in order. Stop at the first that resolves.

1. **An explicit statement wins.** If the user names a model, a vendor, or an API ("this goes to Gemini 3.1 Pro", "for our GPT-6 pipeline"), route on that and ignore the host. The prompt may be written in one agent and executed in another; the destination is what matters, not where the dialogue happens.
2. **Otherwise infer from the host.** Use the map below. Single-vendor hosts resolve with no further work.
3. **Otherwise use `universal-baseline.md`.** A multi-model host with no visible selection is not a reason to guess: the baseline is the vendor-agnostic intersection and is a correct answer, not a degraded one.

**Do not spend a tier-capped dialogue question on the runtime.** It is a routing decision, not a component — the caps in `SKILL.md` > Step 2 exist for the 7 components. When the target is genuinely load-bearing and unresolved (complex tier, or the prompt will be reused across runtimes), state the assumption in one line while presenting the prompt in Step 3, phrased so the user can correct it without a round trip: "Written for Claude Opus 5.5 (XML tags, no self-check clause) — say the word if it is going somewhere else."

**Never infer a vendor from a multi-model host.** Writing XML-heavy Claude structure for a Gemini-backed session, or carrying a `temperature: 0.2` instruction into a Gemini 3.x prompt whose vendor says to remove sampling parameters, is a measurable regression, not a stylistic difference.

## Host → vendor map (verified 2026-09-25)

| Host | Model family | Load | Kind |
|---|---|---|---|
| Claude Code (CLI, desktop Code tab, IDE extensions) | Anthropic Claude (Opus 5.5 by default on paid plans and the API) | `claude-considerations.md`, then the per-model file | Single-vendor |
| Codex (CLI, desktop app, VS Code Codex extension) | OpenAI GPT (GPT-6 Sol recommended) | `codex-considerations.md` | Single-vendor |
| Gemini CLI | Google Gemini. Replaced by Antigravity CLI for unpaid-tier and Google One users on 2026-06-18; still available with paid API keys | `gemini-considerations.md` | Single-vendor |
| Jules | Google Gemini | `gemini-considerations.md` | Single-vendor |
| Antigravity CLI (`agy`) / Antigravity IDE | Google Gemini first (3.8 / 3.7 / 3.6 Flash, 3.1 Pro); Claude and GPT-OSS models are also in the picker | `gemini-considerations.md` unless the picker says otherwise | Gemini-first, switchable |
| Grok Build | xAI Grok (`grok-4.7` by default); custom models, including other vendors', can be configured | `grok-considerations.md` unless the config names another model | Default known, switchable |
| GitHub Copilot (CLI, editor extensions, cloud agent) | Vendor picker: Anthropic, OpenAI, Google, xAI, Microsoft, Moonshot AI | The vendor file for the selected model, else `universal-baseline.md` | Host-selected |
| Cursor, Windsurf (renamed Devin Desktop), Cline, Zed, Kiro, Qoder, OpenCode, Aider, Amp, Junie, Devin, pi, Swival, OpenClaw, CodeWhale | Host- or user-selected, frequently changed | The vendor file if the selection is visible, else `universal-baseline.md` | Host-selected |
| API integration, CI job, unknown harness | Unknown | `universal-baseline.md` | — |

**Reading the selection on a switchable or host-selected runtime**, cheapest first: the model named in the session UI or statusline; the model the user mentioned earlier in the conversation; a config file already in context (`~/.grok/config.toml`, a `model` key in the host's settings, a note in `AGENTS.md`). If none of those is already available, stop — do not spend a tool call to discover it on a quick- or standard-tier prompt. Route to the host's default, or to `universal-baseline.md` on a host-selected runtime.

This map is the most perishable content in the skill: hosts add vendors, switch defaults, and get renamed. Treat a row as stale if the host's own documentation disagrees with it, and follow the update procedure below.

## Cross-vendor divergence

Where the vendors actively disagree. Applying the wrong column is not neutral — each cell below is something a vendor states about its own models.

| Behavior | Anthropic Claude | OpenAI GPT-6 | Google Gemini 3.x | xAI Grok 4.7 |
|---|---|---|---|---|
| Structure format | XML tags are the strongest tool | Markdown headings or XML, no preference | Clear delimiters — XML-style tags **or** Markdown headings | Not specified for sections; **native tool calling instead of XML tool-call output** |
| Prompt size | Lean; delete legacy scaffolding | Lean; leaner system prompts scored ~10–15% higher with 41–66% fewer tokens on GPT-5.6 ([source](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6)); GPT-6 is more sensitive to conflicting context | "Be concise" | **Detailed system prompt recommended** — the one vendor that asks for more, not less |
| Examples | First component to cut; one complete example is the documented fix for unmarked quotations on Fable 5.1 | Only when one example changes behavior | Recommended to regulate format; drop the chain-of-thought kind | Concrete, specific context over generic examples |
| Reasoning depth | `effort` parameter (Opus 5.5 default `medium`); never "think hard" in text, and drop "think carefully" lines on Opus 5.5 | `reasoning.effort` (`medium` default on Sol and Luna; no `none` on Astra); never in text | `thinking_level` (`medium` default on 3.5 and 3.8 Flash); **and** "Think very hard before answering" can help on heavy-reasoning problems | `reasoning_effort` (`low`…`xhigh`, default `high`), cannot be disabled |
| Verbosity and formatting | Length must be prompted on Opus 5.x; Fable 5.1 formats less and writes fewer progress updates — ask for them | Defaults to lists, tables, and Markdown — state the writing style; `text.verbosity` for detail | Terse by default; ask explicitly for a conversational or longer answer | Not documented |
| Self-check clause | **Remove** on Opus 5 / 5.5; keep on Fable long runs | Tests thoroughly by default — scope verification *down* on small changes | 3.8 Flash verifies its work by design; use Search grounding and code execution for accuracy | Not documented |
| Autonomy on unattended runs | Opus 5.5 and Fable 5.1 can end a turn early — name the early stops, tell it the user is not watching | Asks more clarifying questions — state that the request authorizes the work | Define when to assume and when to ask | Not documented |
| Subagent delegation | **Cap** on Opus 5 / 5.5; encourage on Fable | Delegates less than desired — say when to delegate | Not documented | Not documented |
| Sampling parameters | Not a prompt concern; non-default values return a 400 on Fable 5.x | Removed from requests when reasoning is on | **Remove from all requests**; changing them can cause looping | Not documented |
| Context placement | Long data at the top, query at the end — queries at the end "can improve response quality by up to 30 percent in tests" ([source](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting)) | Domain context near the top | Data first, instructions and questions last, with an anchor phrase | Name the specific files, paths, and dependencies; exclude irrelevant context |
| Caching | Not a prompt-text concern | Explicit prompt-prefix caching | Not a prompt-text concern | `prompt_cache_key` strongly recommended for repeated prefixes |

Three rows deserve emphasis because they invert the usual advice:

- **xAI asks for detail where Anthropic, OpenAI, and Google ask for brevity.** A prompt trimmed for Gemini 3.x can underperform on Grok. Do not carry the lean-prompt reflex across that boundary without checking `grok-considerations.md`.
- **Google endorses a reasoning nudge in prompt text** ("Think very hard before answering") that Anthropic and OpenAI replace with a parameter. This is the clearest case where copying a Claude prompt to Gemini leaves quality on the table, and vice versa.
- **Delegation and verification run in opposite directions on the two current flagships.** Opus 5.5 needs delegation capped and verification clauses removed; GPT-6 needs delegation requested and its testing scoped down. Copying either constraint across vendors reverses its effect.

## What never goes into a prompt whose vendor is unknown

- Vendor-specific parameter names or values as prose (`effort: high`, `thinking_level: low`, `text.verbosity`, `temperature: 0.2`). They are API fields, not instructions, and one vendor's safe value is another's regression.
- Assistant prefill, or any assumption that the last assistant turn can be seeded — unsupported on current Claude models and removed from Gemini 3.8 Flash requests.
- A tool-call output format ("reply with `<tool_call>` XML"), which conflicts with native tool calling on xAI and with structured tool APIs generally.
- "Reproduce your reasoning in the response", a refusal trigger on Claude Fable 5.x and Opus 5.5 and pointless elsewhere.
- Any claim about a context window, cutoff date, or model name. If the prompt names the model, it stops being portable.

## Update procedure

1. **Re-verify the host map first** — it rots faster than the vendor files. For every `Single-vendor` row, confirm the host has not added a picker or custom-model support; for every `Host-selected` row, confirm the host still exists under that name.
2. When a vendor ships a new generation, update its own file (`claude-considerations.md` and its per-model files, `codex-considerations.md`, `gemini-considerations.md`, `grok-considerations.md`) and then **re-diff the divergence table** above. New generations tend to invert prior guidance rather than extend it.
3. Add a vendor file only when a host in the map routes to it. A file no host reaches is dead content; a host with no file is a gap that silently falls back to the baseline.
4. Update the verification record in `maintenance.md` with the date and the pages checked, and re-run fixture set H in `tests/activation-fixtures.md`.
