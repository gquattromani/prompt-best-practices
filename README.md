# Prompt Best Practices

An AI agent skill that transforms unstructured prompts into high-quality structured prompts through a short interactive dialogue. Works with any AI agent that supports the [Agent Skills](https://agentskills.io/) open standard, and ships a thin adapter for every major agent platform — plugin manifests where the host installs plugins, an always-on rule file where it only reads project instructions ([full mapping](docs/agent-portability.md)).

**Runtime-aware: the prompt is composed for the model that will consume it.** The 7-component framework was built by studying the prompting guidelines published by the major LLM providers. The canonical reference is [Anthropic's prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) — the most comprehensive publicly available guide on the subject — with each component mapped to a specific section of that documentation. The framework then maps onto [OpenAI's GPT-6 guidance](https://developers.openai.com/api/docs/guides/latest-model/gpt-6-astra), [Google's Gemini 3.x guidance](https://ai.google.dev/gemini-api/docs/latest-model), and [xAI's Grok documentation](https://docs.x.ai/developers/grok-4-7), with dedicated tuning notes per runtime under `skills/prompt-best-practices/references/`.

Which notes apply is decided by **where the skill is running**: Claude Code resolves to Anthropic, Codex to OpenAI, Gemini CLI / Antigravity / Jules to Google, Grok Build to xAI. On a multi-model host (Copilot, Cursor, Windsurf, Cline, Zed, Kiro, Qoder, OpenCode, Aider, Amp, Junie, Devin) the skill does not guess — it builds to a **universal baseline**, the intersection every vendor endorses, and says which runtime it assumed. A model you name yourself always outranks the host, because prompts are often written in one agent and executed in another.

Baselines are current (verified 2026-09-25): **Claude Opus 5.5** as the default Anthropic target, Claude Fable 5.1 / Mythos 5.1 as the highest-capability tier, OpenAI GPT-6 (Sol in Codex, Astra as the most capable), Google Gemini 3.x with 3.8 Flash current, and xAI `grok-4.7`. Guidance is **not additive**: an instruction that helps one runtime measurably hurts another — a self-check clause is recommended on Fable long runs and must be *removed* on Opus 5.5; subagent delegation is capped on Opus 5.5 and requested on GPT-6; three vendors reward leaner prompts while xAI asks for a thorough one; Google endorses a "think very hard" nudge that Anthropic and OpenAI replace with a parameter. Every one of those conflicts is recorded in an explicit divergence table rather than averaged away.

## The Problem

Most prompts sent to AI agents lack the structure needed to produce reliable, high-quality output. Users provide a vague goal — "write me an email", "build this feature" — without defining success criteria, output format, constraints, or context. The model fills in the blanks with assumptions, and what follows is multiple rounds of corrections to converge on what the user actually wanted.

This skill reduces that cycle. It intercepts underspecified prompts and guides you through a short, tier-adaptive dialogue (up to 2 questions for quick tasks, up to 5 for complex ones) to surface the missing information. The result is a structured prompt sized to the task weight.

## Non-goals

- **Not a guarantee** of better output. The framework encodes well-documented heuristics but makes no empirical claim without a benchmark run. See `tests/benchmark-protocol.md` for how to measure effect size on your workload.
- **Not a generic writing coach.** It skips questions, micro-tasks, and conversational exchanges. See `tests/activation-fixtures.md` for the exact contract.
- **Not XML-only.** Structure is format-agnostic (XML, Markdown, JSON all qualify). XML is the Claude default.
- **Not a framework factory.** The 7 components are the framework. PRs adding new components belong in a fork. See [CONTRIBUTING.md](CONTRIBUTING.md) > Scope and non-goals.

## Demo

<p align="center">
  <img src="assets/demo.gif" alt="Prompt Best Practices skill demo" width="800">
</p>

## Installation

No configuration and no dependencies. Every platform loads the same skill, either as a plugin (skill + the `/prompt-best-practices` command) or as an always-on rule file. Which file each host reads: [docs/agent-portability.md](docs/agent-portability.md).

Plugin-tier hosts that support hooks also get a one-line `UserPromptSubmit` reminder so the skill actually fires without the slash command — see [Automatic activation](#automatic-activation) for what it does and how to turn it off. It is a single shell `echo`: no `node`, no scripts, no state.

### Any compatible agent, via `skills.sh`

```bash
npx skills add gquattromani/prompt-best-practices -g -y
```

Installs the skill globally — available in every project, in every agent that supports the [Agent Skills](https://agentskills.io/) standard.

### Claude Code

```
/plugin marketplace add gquattromani/prompt-best-practices
```

```
/plugin install prompt-best-practices@prompt-best-practices
```

Send the two commands as **separate messages** — the install does not work if they are combined.

The same two commands work in the Claude Code desktop app's Code tab: type them into the prompt box, or click the **+** button next to it, choose **Plugins** → **Add plugin** to browse your configured marketplaces, and manage marketplaces from **Customize** in the sidebar.

Verify with `/plugin` (or `claude plugin list`). Scope the install with `claude plugin install prompt-best-practices@prompt-best-practices --scope project` if you want it committed to a single repository instead of your user profile.

The plugin registers one `UserPromptSubmit` hook (a shell `echo`, no `node`). Review it with `/hooks`; disabling it there keeps the skill and the slash command, and only turns off automatic activation.

### Codex

```bash
codex plugin marketplace add gquattromani/prompt-best-practices
codex plugin add prompt-best-practices@prompt-best-practices
```

Then run `codex`, open `/hooks` to review and trust the single `UserPromptSubmit` hook (a shell `echo` that reminds the agent to assess an underspecified request), and start a new thread. Declining the hook keeps the skill and the command; only automatic activation goes away.

The same install covers the Codex desktop app: restart the app afterwards and it picks up the plugin.

Instruction-only fallback, no plugin: Codex reads `AGENTS.md` from the repository root, or `~/.codex/AGENTS.md` globally.

### GitHub Copilot CLI

```bash
copilot plugin marketplace add gquattromani/prompt-best-practices
copilot plugin install prompt-best-practices@prompt-best-practices
```

In an interactive Copilot CLI session, use the slash equivalents:

```
/plugin marketplace add gquattromani/prompt-best-practices
/plugin install prompt-best-practices@prompt-best-practices
```

The plugin also registers a `userPromptSubmitted` hook for automatic activation; remove the `hooks` key from `.github/plugin/plugin.json` to opt out.

Copilot CLI namespaces plugin commands by plugin name:

```text
/prompt-best-practices:prompt-best-practices
```

Instruction-only fallback, no plugin: it reads `AGENTS.md` and `.github/copilot-instructions.md` in a project, or copy the rule into `~/.copilot/copilot-instructions.md` to run it in every project. That path keeps the always-on rule but not the slash command.

### Gemini CLI

```bash
gemini extensions install https://github.com/gquattromani/prompt-best-practices
```

Loads `AGENTS.md` as always-on context every session and registers `/prompt-best-practices`; `skills/` ships along and activates when a task needs it. Verify with `gemini extensions list`.

No root `hooks/hooks.json` is shipped: Gemini auto-loads that exact path, and the hook maps in `hooks/` use Claude and Codex event names. Gemini needs no hook anyway — it loads `AGENTS.md` as always-on context, so the activation rule is already in front of the model.

### Antigravity CLI (`agy`)

Antigravity installs a plugin from a local directory, so clone it first:

```bash
git clone https://github.com/gquattromani/prompt-best-practices
agy plugin install ./prompt-best-practices
```

The plugin root carries `plugin.json`; `skills/` and `rules/` are read from it, so the always-on rule ships as [`rules/prompt-best-practices.md`](rules/). Installing from a repository URL is not documented, which is why the clone step is explicit. Antigravity also recognises `.agents/skills/` for workspace-specific skills.

### Grok Build

```bash
grok plugin install gquattromani/prompt-best-practices --trust
```

Plugins are off by default — enable it with `/plugins` → Plugins → Space on `prompt-best-practices`, or in `~/.grok/config.toml`:

```toml
[plugins]
enabled = ["prompt-best-practices"]
```

Start a new session (or reload plugins), then verify with `grok inspect`. The skill shows as `/prompt-best-practices`; Grok can also auto-invoke it from the skill description when a prompt needs structuring.

`AGENTS.md` still works instruction-only from a checkout, without the plugin.

### Qoder

`AGENTS.md` is auto-loaded from the repository root, so running Qoder from a checkout works with zero setup — the always-on rule already carries the activation contract. For per-project rules, copy [`.qoder/rules/prompt-best-practices.md`](.qoder/rules/) into your project's `.qoder/rules/`. The plugin manifest [`.qoder-plugin/plugin.json`](.qoder-plugin/plugin.json) points at `skills/`; `.qoder/rules/` is read by convention rather than declared, because Qoder's manifest has no `rules` field. Qoder registers hooks from its own settings file rather than from a plugin manifest, so no hook file ships for it; if you want the activation reminder there too, the snippet to paste into `.qoder/settings.json` is in [`docs/agent-portability.md`](docs/agent-portability.md) under Activation delivery.

### OpenCode

`AGENTS.md` is auto-loaded from the repository root, so a checkout needs no setup. For the explicit slash command, copy [`.opencode/commands/prompt-best-practices.md`](.opencode/commands/) into your project's `.opencode/commands/`, or into `~/.config/opencode/command/` to have it in every project.

No server plugin ships — OpenCode plugins exist to inject instructions each turn, which this skill does not need.

### pi

```bash
pi install git:github.com/gquattromani/prompt-best-practices
```

Registers `skills/` through the `pi` field in `package.json`.

### Swival

Stage the collection in your library first, then add it where you want it:

```bash
swival skills add --global https://github.com/gquattromani/prompt-best-practices  # stage into ~/.config/swival/library
swival skills add prompt-best-practices                                          # this project
swival skills add --global prompt-best-practices                                 # every project
```

On the command line, a `$` prefix activates a skill explicitly: `$prompt-best-practices`. Swival also reads `AGENTS.md` from the project root and `~/.config/swival/AGENTS.md` globally, as the instruction-only fallback.

### OpenClaw

```bash
cp -R skills/prompt-best-practices ~/.openclaw/skills/
```

Copy the **folder**, not the single `SKILL.md` — the skill ships its own `references/`, which is why no flattened copy is kept in this repository.

### CodeWhale

Reads `AGENTS.md` from the project root, zero setup: copy [`AGENTS.md`](AGENTS.md) into your project, or run `codewhale` from a checkout of this repository. It also falls back to `CLAUDE.md` and `.claude/instructions.md`.

### Hermes Agent

No native plugin ships: the Hermes plugin format expects a Python entry point registering hooks and commands, and this skill has neither. Use the instruction-only path — copy [`AGENTS.md`](AGENTS.md) into the project — or copy `skills/prompt-best-practices/` into the location your Hermes install reads skills from.

### Aider

Aider takes the rule as read-only context:

```bash
aider --read AGENTS.md
```

Or make it permanent in `.aider.conf.yml`:

```yaml
read: AGENTS.md
```

### Cursor, Windsurf, Cline, Kiro, Copilot in the editor, Zed, Amp, Jules, Junie

These hosts read project instructions only, so they get the compact always-on rule — activation contract, task tiers, the 7 components, dialogue caps — instead of the full reference set. Copy the matching file into your project:

| Host | Project file | Every project |
|---|---|---|
| Cursor | [`.cursor/rules/prompt-best-practices.mdc`](.cursor/rules/) | `~/.cursor/rules/` |
| Windsurf | [`.windsurf/rules/prompt-best-practices.md`](.windsurf/rules/) | `~/.codeium/windsurf/memories/global_rules.md` |
| Cline | [`.clinerules/prompt-best-practices.md`](.clinerules/) | Cline settings → Custom Instructions |
| Kiro | [`.kiro/steering/prompt-best-practices.md`](.kiro/steering/) | `~/.kiro/steering/` |
| GitHub Copilot (VS Code / JetBrains / Visual Studio extension) | [`.github/copilot-instructions.md`](.github/copilot-instructions.md) | `~/.copilot/copilot-instructions.md` |
| Zed | [`AGENTS.md`](AGENTS.md), auto-included from the worktree root | `~/.config/zed/` rule files |
| Amp (Sourcegraph) | [`AGENTS.md`](AGENTS.md), read from the working directory up to `$HOME` | `~/.config/amp/AGENTS.md` |
| Jules (Google) | [`AGENTS.md`](AGENTS.md), read automatically from the repository root | — |
| VS Code + Codex extension | [`AGENTS.md`](AGENTS.md), read from the repository root | `~/.codex/AGENTS.md` |
| JetBrains Junie | [`AGENTS.md`](AGENTS.md), pointed at in Settings → Tools → Junie → Project Settings → Guidelines Path (not automatic yet; `.junie/guidelines.md` is the legacy path) | — |

Every one of those files is a byte-identical copy of the canonical rule body in `AGENTS.md`, verified by fixture G1 in [`tests/activation-fixtures.md`](tests/activation-fixtures.md).

## Usage

```
/prompt-best-practices
```

The skill activates automatically when you ask to write, draft, generate, implement, fix, refactor, or build any output that lacks clear structure. Invoke it explicitly to force activation on any prompt — assessment is then mandatory, and you can still exit at the first message with `skip`, `go`, `execute`, or `as-is`.

How it is invoked depends on the host:

| Host | Invocation |
|---|---|
| Claude Code, Codex, Gemini CLI, Grok Build, Qoder | `/prompt-best-practices` |
| GitHub Copilot CLI | `/prompt-best-practices:prompt-best-practices` |
| Antigravity CLI | `/prompt-best-practices` typed as a chat message (commands become skills) |
| OpenCode | `/prompt-best-practices`, once the command file is in `.opencode/commands/` |
| Swival | `$prompt-best-practices` |
| Instruction-tier hosts (Cursor, Windsurf, Cline, Kiro, Copilot in the editor, Zed, Amp, Jules, Junie, CodeWhale, Aider) | No command — the rule is always on, so ask for the work and the assessment runs on underspecified prompts |

### Automatic activation

Before a skill is loaded, an agent sees only its frontmatter `description` — the `AUTO-ACTIVATE when…` rules inside `SKILL.md` are invisible until the skill has already been chosen. Add the structural bias that assessing means interrupting a request the agent would rather just execute, and a skill like this one fires rarely on its own. Two mechanisms fix it, and which one you get depends on the host:

| Host | How activation is delivered |
|---|---|
| Cursor, Windsurf, Cline, Kiro, Copilot in the editor, Zed, Amp, Jules, Junie, CodeWhale, Aider | The rule file is always in context, so the activation contract is present before the agent decides. Nothing to configure. |
| Gemini CLI, Antigravity, OpenCode, Swival, Qoder | `AGENTS.md` is auto-loaded from the repository root — same effect. |
| Claude Code, Codex, Copilot CLI | A `UserPromptSubmit` hook prints one line of reminder into that turn's context. This is the only documented way for a plugin to put the rule in front of the model before it decides. |
| Grok Build, pi, OpenClaw | Model discretion from the `description` alone. Copy [`AGENTS.md`](AGENTS.md) into the project if you want it always on. |

The hook is a reminder, not a gate: it repeats the skill's own skip conditions (micro-tasks, questions, an explicit "just do it"), so the agent still decides and a one-word `skip` still exits. Turn it off with `/hooks` on Claude Code or Codex, or by removing the `hooks` key from the host's manifest — the skill and the slash command keep working.

Prefer no hook at all? Put the compact rule in your own always-on config instead — `~/.claude/CLAUDE.md` for every project, or a project `CLAUDE.md`:

```markdown
## Prompt structuring
Before acting on an execution request that does not state its success criteria,
output format, or constraints, invoke the prompt-best-practices skill first.
Skip for micro-tasks, questions, and explicit "just do it" requests.
```

## How It Works

The skill evaluates your prompt against 7 components, each grounded in a specific section of [Anthropic's prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices):

| # | Component | What it checks | Source |
|---|---|---|---|
| 1 | **Task** | Clear action with measurable success criteria | [Be clear and direct](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#be-clear-and-direct) |
| 2 | **Role** | Defined expertise area or persona | [Give a role](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#give-claude-a-role) |
| 3 | **Context** | Relevant documents or data | [Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting) |
| 4 | **Examples** | Concrete examples of desired output | [Use examples effectively](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#use-examples-effectively) |
| 5 | **Output specification** | Format, tone, audience, anti-patterns | [Control the format of responses](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#control-the-format-of-responses) |
| 6 | **Constraints** | Rules with motivation (why each exists) | [Add context to improve performance](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#add-context-to-improve-performance) |
| 7 | **Structure** | Tagged sections for unambiguous parsing | [Structure prompts with XML tags](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#structure-prompts-with-xml-tags) |

Components are classified as present / partial / missing using the rubric in `skills/prompt-best-practices/references/component-rubrics.md`, so the diagnostic is reproducible across runs.

The dialogue length is capped by task tier (`skills/prompt-best-practices/references/framework.md` > Task Tiers): up to 2 questions for quick tasks, 3 for standard, 5 for complex. If you prefer to skip the dialogue entirely, reply `skip` or `go` to the first diagnostic message.

### Composed for the runtime it will run on

The components are the same on every runtime. The dialect is not — and the differences are not cosmetic:

| Target | How the final prompt is composed |
|---|---|
| **Anthropic Claude** (Claude Code) | XML tags; long data at the top, task at the end; self-check and verification clauses **removed** on Opus 5.5 (kept on Fable long runs); response length stated explicitly, because `effort` does not shorten visible output. |
| **OpenAI GPT** (Codex) | Markdown headings or XML; repeated instructions and non-behavioral examples dropped; one authorization boundary that, on GPT-6, also grants the autonomy the request implies; writing style stated; the `apply_patch` tool when the task edits files. |
| **Google Gemini** (Gemini CLI, Antigravity, Jules) | Markdown or XML used consistently; data first, instruction last, anchored back to the data; length and tone stated explicitly because Gemini 3.x is terse by default; **no sampling instruction** — the vendor says to remove sampling parameters from requests; a "think very hard" nudge only on heavy-reasoning tasks, where Google endorses it. |
| **xAI Grok** (Grok Build) | The one runtime that asks for a *thorough* prompt: Role kept, edge cases spelled out in Constraints, context paths enumerated; tool use described as native function calling, never as an XML envelope. |
| **Unresolved / multi-model host** | The universal baseline: XML tags (the hidden model may be Claude, and no other vendor penalizes them), context first, one example at most, explicit length, motivated rules — and none of the vendor-specific parameter text (`temperature`, `effort`, `thinking_level`, `verbosity`) that is correct on one runtime and wrong on another. |

Host-to-vendor routing, the divergence table, and the baseline live in `skills/prompt-best-practices/references/runtime-detection.md` and `universal-baseline.md`. Fixture set H in [`tests/activation-fixtures.md`](tests/activation-fixtures.md) is the regression contract for it.

## Compatible Platforms

This skill follows the [Agent Skills](https://agentskills.io/) open standard and ships a thin adapter for every agent platform that supports plugins, skills, or project instructions — Claude Code, Codex, GitHub Copilot CLI, Gemini CLI, Antigravity, Grok Build, Qoder, OpenCode, pi, Swival, OpenClaw, Cursor, Windsurf, Cline, Kiro, Zed, Amp, Jules, Junie, CodeWhale, Aider. The full mapping of which file each host reads, and what is deliberately not shipped, is in [docs/agent-portability.md](docs/agent-portability.md).

The 7-component framework applies to every runtime; what varies is the structural format (Component 7) and a handful of runtime-specific constraints. Model-specific tuning notes:

- `skills/prompt-best-practices/references/runtime-detection.md` — the **top-level router**: host → model family, the cross-vendor divergence table, and what never goes into a prompt whose vendor is unknown.
- `skills/prompt-best-practices/references/universal-baseline.md` — the vendor-agnostic intersection, plus the delta to apply once the vendor becomes known.
- `skills/prompt-best-practices/references/claude-considerations.md` — Anthropic Claude **router**: which per-model file to load, the behaviors shared across the family (XML tags, no prefill, effort instead of thinking budgets), and the per-model divergence table.
- `skills/prompt-best-practices/references/claude-opus-5-5.md` — Claude Opus 5.5, the default target, and Claude Opus 5. Prompt explicitly for conciseness (`effort` does not shorten visible output); delete verification and self-check clauses; constrain scope; cap subagent delegation; on Opus 5.5, thinking is always on, "think carefully" lines go, and unattended runs name the early stops to avoid.
- `skills/prompt-best-practices/references/claude-fable-5.md` — Claude Fable 5.1 / Mythos 5.1 and Fable 5 / Mythos 5, the highest-capability tier. Refactor rather than over-prescribe; effort is the primary dial; steer with brief instructions; ground progress claims on long runs; on 5.1, ask for progress updates and delete anti-formatting rules.
- `skills/prompt-best-practices/references/codex-considerations.md` — OpenAI Codex (GPT-6 Sol / Astra / Luna; GPT-5.6 previous generation). Markdown headings or XML both work; outcome-first / leaner prompts; an authorization boundary that grants autonomy on GPT-6; explicit writing style; test scope instead of verification clauses; the `apply_patch` tool for edits.
- `skills/prompt-best-practices/references/gemini-considerations.md` — Google Gemini (Gemini 3.x, 3.8 Flash current). Concise input, terse output by default, sampling parameters removed, `thinking_level` for depth (`medium` default), data-first context, examples kept for format regulation, grounding via Search and code execution.
- `skills/prompt-best-practices/references/grok-considerations.md` — xAI Grok (`grok-4.7`, the default model of Grok Build). Thorough system prompt with edge cases, enumerated context paths, `reasoning_effort` for depth, native tool calling over XML tool-call output, `prompt_cache_key` for reused prefixes.

Anthropic, OpenAI and Google all reward leaner prompts, so the framework aims for the *smallest sufficient* prompt rather than the most complete one — with xAI as the documented exception, and with the vendors disagreeing on which sections to drop even where they agree on the direction. That is what the divergence table is for. The framework is most thoroughly tested on Claude because its components map cleanly onto Anthropic's published guide. Reports on other runtimes are welcome — see `tests/benchmark-protocol.md` to run a comparable evaluation.

## Uninstall

The uninstall procedure depends on how you installed the skill.

### If installed via Claude Code plugin

```bash
claude plugin uninstall prompt-best-practices
```

Scope flags (optional):

```bash
claude plugin uninstall prompt-best-practices --scope user       # global (default)
claude plugin uninstall prompt-best-practices --scope project    # project-level
claude plugin uninstall prompt-best-practices --scope local      # local only
```

Verify with `claude plugin list`.

**Manual fallback (Claude Code):** delete the `prompt-best-practices` entry from the `enabledPlugins` section of the relevant settings file — `~/.claude/settings.json` (user-global), `.claude/settings.json` (project-shared), or `.claude/settings.local.json` (local, git-ignored).

### Other hosts

| Host | Command |
|---|---|
| Codex | `codex plugin remove prompt-best-practices` |
| GitHub Copilot CLI | `copilot plugin uninstall prompt-best-practices` |
| Grok Build | `grok plugin uninstall prompt-best-practices` |
| Gemini CLI / Antigravity | `gemini extensions uninstall prompt-best-practices` |
| pi | `pi uninstall prompt-best-practices` |
| Swival | `swival skills remove prompt-best-practices` |
| OpenClaw | `rm -rf ~/.openclaw/skills/prompt-best-practices` |
| Cursor / Windsurf / Cline / Kiro / Qoder / Copilot in the editor / Aider | Delete the copied rule file |

The skill writes no state outside its own install folder, so removing it from the host's plugin or skill registry is the whole teardown.

### If installed via `skills.sh`

Follow the uninstall procedure documented by your installer or agent. The skill itself requires no special teardown — removing it from the agent's plugin/skill registry is sufficient.

## Repository Structure

```
.
├── .claude-plugin/          Claude Code plugin + marketplace manifest
├── .codex-plugin/           Codex plugin manifest
├── .qoder-plugin/           Qoder plugin manifest (points at skills/ and .qoder/rules/)
├── .grok-plugin/            Grok Build marketplace manifest (with root plugin.json)
├── .github/
│   ├── plugin/              GitHub Copilot CLI plugin + marketplace manifest
│   └── copilot-instructions.md   Copilot editor-extension rule
├── rules/                   Antigravity plugin rule copy
├── .cursor/ .windsurf/ .clinerules/ .kiro/ .qoder/
│                            Instruction-tier rule copies (see docs/agent-portability.md)
├── .opencode/commands/      OpenCode slash command
├── commands/                Slash command (TOML) for Codex, Gemini CLI, Grok, Copilot CLI
├── hooks/                   UserPromptSubmit activation reminders (Claude/Codex, Copilot)
├── gemini-extension.json    Gemini CLI / Antigravity extension manifest
├── plugin.json              Grok Build root manifest
├── package.json             pi harness manifest (skills registration)
├── assets/                  Media files
├── docs/
│   └── agent-portability.md Which file each agent platform reads
├── skills/
│   └── prompt-best-practices/
│       ├── SKILL.md         Skill entry point
│       └── references/      Framework entry point, component definitions, rubrics,
│                            runtime detection (host → vendor), universal baseline,
│                            per-vendor tuning notes (Claude, Codex, Gemini, Grok),
│                            examples, maintenance record
├── tests/
│   ├── activation-fixtures.md   Reference prompts with expected activation outcome,
│   │                            plus mechanical loadability (F) and portability (G) checks
│   └── benchmark-protocol.md    Protocol to measure framework effect size
├── AGENTS.md                Canonical always-on rule (above the marker) + repository map
├── CHANGELOG.md             Version history
├── CODE_OF_CONDUCT.md       Contributor Covenant
├── CONTRIBUTING.md          Contribution guidelines
├── SECURITY.md              Vulnerability reporting policy
├── NOTICE.md                Third-party attributions and trademark acknowledgments
└── LICENSE                  MIT
```

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for the process and [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for the community standard. For security issues, see [SECURITY.md](SECURITY.md) — please do not open a public issue for vulnerabilities.

## License

MIT — see [LICENSE](LICENSE).

Third-party attributions, the licence basis for every quotation, and the full trademark acknowledgment are in [NOTICE.md](NOTICE.md). Every product name used in this repository belongs to its owner; this project is independent and is not affiliated with or endorsed by any of them.
