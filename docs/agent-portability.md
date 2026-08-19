# Agent Portability

This repository is an agent-portable skill distribution. `skills/prompt-best-practices/`
holds the behavior; every host-specific file is a thin adapter that makes that
behavior loadable in a given agent.

Two tiers of support:

- **Plugin-tier** — the host installs the repository as a plugin/extension and gets the skill plus the `/prompt-best-practices` command.
- **Instruction-tier** — the host only reads project instructions, so it gets the compact always-on rule (activation contract, tiers, 7 components, dialogue caps) instead of the full workflow.

## Supported adapters

| Host | Files | Tier | Notes |
|---|---|---|---|
| Claude Code | `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `skills/` | Plugin | `/plugin marketplace add` + `/plugin install`. The skill auto-activates from its `description` and is invocable as `/prompt-best-practices`. |
| Codex | `.codex-plugin/plugin.json`, `skills/`, `commands/` | Plugin | Manifest points `skills` at `./skills/`; `commands/*.toml` is auto-discovered. Same install covers the Codex desktop app. |
| GitHub Copilot CLI | `.github/plugin/plugin.json`, `.github/plugin/marketplace.json`, `commands/`, `skills/` | Plugin | Commands are namespaced by plugin name (`/prompt-best-practices:prompt-best-practices`). Fallback: `.github/copilot-instructions.md` per project or `~/.copilot/copilot-instructions.md` globally. |
| Grok Build | root `plugin.json`, `.grok-plugin/marketplace.json`, `skills/`, `commands/` | Plugin | `grok plugin install <repo> --trust`, then enable it (plugins are off by default). Grok can auto-invoke the skill from its description; the command makes it explicit. |
| Gemini CLI | `gemini-extension.json`, `AGENTS.md`, `commands/`, `skills/` | Plugin | `contextFileName` points at `AGENTS.md` for always-on rules; `commands/*.toml` and `skills/` are auto-discovered. |
| Antigravity CLI (`agy`) | root `plugin.json`, `rules/`, `skills/` | Plugin | `plugin.json` at the plugin root is the documented manifest (`name` required, `description` optional); the rule copy lives in `rules/` and the skill in `skills/`, both documented members of the plugin layout. Install from a local checkout — `agy plugin install <path>`; a repository URL is not documented. |
| Qoder | `.qoder-plugin/plugin.json`, `.qoder/rules/`, `skills/`, `AGENTS.md` | Plugin | Manifest points at `skills/`; `.qoder/rules/` is read by convention, not declared (Qoder's manifest has no `rules` field). `AGENTS.md` is auto-loaded from the repo root, so a checkout works with zero setup. Qoder registers hooks from its own settings file, not from a plugin manifest — see the snippet under "Activation delivery". |
| OpenCode | `.opencode/commands/`, `skills/`, `AGENTS.md` | Plugin | `AGENTS.md` is auto-loaded from the repo root; the command file adds the explicit slash command. No server plugin ships — see "Not shipped" below. |
| Swival | `skills/`, `AGENTS.md` | Plugin | `swival skills add https://github.com/gquattromani/prompt-best-practices` (add `--global` to stage in the library first). Also reads `AGENTS.md` from the project root and `~/.config/swival/AGENTS.md`. |
| OpenClaw | `skills/` | Plugin | Copy `skills/prompt-best-practices/` into `~/.openclaw/skills/`. The skill folder ships its `references/`, so no flattened copy is kept in the repo. |
| Cursor | `.cursor/rules/prompt-best-practices.mdc` | Instruction | Always-on project rule (`alwaysApply: true`). |
| Windsurf | `.windsurf/rules/prompt-best-practices.md` | Instruction | Project rule. |
| Cline | `.clinerules/prompt-best-practices.md` | Instruction | Project rule. |
| Kiro | `.kiro/steering/prompt-best-practices.md` | Instruction | Steering rule (`inclusion: always`); copy into a project or `~/.kiro/steering/`. |
| GitHub Copilot (editor extensions) | `.github/copilot-instructions.md` | Instruction | Repository instruction file, read by the VS Code / JetBrains / Visual Studio extension. |
| Zed | `AGENTS.md` | Instruction | Auto-included from the worktree root as a default rule file. |
| Amp (Sourcegraph) | `AGENTS.md` | Instruction | Read from the working directory upward to `$HOME`; `~/.config/amp/AGENTS.md` works globally. |
| Jules (Google) | `AGENTS.md` | Instruction | Read automatically from the repository root. |
| JetBrains Junie | `AGENTS.md` | Instruction | Point Junie at it in Settings → Tools → Junie → Project Settings → Guidelines Path (not automatic). |
| CodeWhale | `AGENTS.md` | Instruction | Read from the project root; also falls back to `CLAUDE.md`. |
| VS Code + Codex extension | `AGENTS.md` | Instruction | Reads the repo-root `AGENTS.md` (or `~/.codex/AGENTS.md` globally). The Codex plugin row above adds the command. |
| pi | `package.json` (`pi.skills`), `skills/` | Plugin | `pi install git:github.com/gquattromani/prompt-best-practices` registers `skills/`. |
| Aider | `AGENTS.md` | Instruction | Passed as read-only context: `aider --read AGENTS.md`, or `read: AGENTS.md` in `.aider.conf.yml`. |
| Generic agents | `AGENTS.md` or `skills/prompt-best-practices/SKILL.md` | Either | Copy the compact rule, or load the skill folder directly. |

## Activation delivery

A skill that never fires is indistinguishable from a skill that is not installed, and the two tiers deliver activation differently.

**Instruction-tier hosts already have it.** They read `AGENTS.md` (or a rule copy) into context on every turn, so the activation contract — when to assess, when to skip — is present before the agent decides anything.

**Plugin-tier hosts do not, by default.** Before a skill is loaded, only its frontmatter `description` reaches the model; the `AUTO-ACTIVATE when…` rules inside `SKILL.md` are invisible until the skill has already been chosen. That is a chicken-and-egg problem, and it is made worse by a structural bias: assessing means interrupting an execution request to ask a question, while the agent's default drive is to execute. In practice the skill fired rarely without the slash command.

Two fixes ship together:

1. **A trigger-dense `description`** — verbs first, positive framing, exclusions kept to one clause, and an explicit note that the assessment costs one message and is skippable in one word, which is what lowers the perceived cost of activating.
2. **A `UserPromptSubmit` hook** for the hosts whose plugin manifest can register one — `hooks/activation-reminder-claude-codex.json` (Claude Code, Codex) and `hooks/activation-reminder-copilot.json` (Copilot CLI). It is one `echo`: no `node`, no script file, no state. Its stdout is added to the context for that turn, which is the only documented way for a plugin to put the activation rule in front of the model before it decides.

The hook is a **reminder, not a gate**: it carries the same skip conditions as the skill (micro-tasks, questions, explicit "just do it"), so the agent still decides. To disable it, review it with `/hooks` on the host, or remove the `hooks` key from the manifest.

Qoder reads hooks from its settings file rather than from a plugin manifest, so no hook file ships for it. If you want the reminder there on top of the always-on `AGENTS.md`, paste this into `.qoder/settings.json` (project) or `~/.qoder/settings.json` (global):

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo 'prompt-best-practices is installed. Before acting on this prompt, check it: if it is an execution request (write, draft, generate, implement, fix, refactor, build, create, design, analyze) at standard or complex tier that does not state its success criteria, output format or constraints, invoke the prompt-best-practices skill first and compose the final prompt for this runtime. Skip silently for micro-tasks, questions, and when the user asks to just do it.'"
          }
        ]
      }
    ]
  }
}
```

Keep the reminder text identical to the shipped hook files, or fixture G5 in `tests/activation-fixtures.md` stops covering it.

Hosts that need neither: Gemini CLI, Antigravity, OpenCode, Swival and Qoder all auto-load `AGENTS.md` from the repository root, so the rule is already always-on there. This is also why no `hooks/hooks.json` exists at the repository root — Gemini CLI auto-loads that exact path, and the file it would find uses Claude and Codex event names. The manifests point at explicitly named hook files instead.

## Prompt dialect per host

An adapter decides *whether* the skill loads on a host. It also decides *what the skill writes*: the host determines which model family will consume the prompt, and the vendors disagree — XML tags on Claude, terse-by-default and no sampling instructions on Gemini, thorough prompts with edge cases on Grok, lean prompts on GPT-5.6.

That mapping is deliberately **not** duplicated here. `skills/prompt-best-practices/references/runtime-detection.md` is the single source of truth: host → model family → which vendor file to load, plus the cross-vendor divergence table and the universal baseline used when a host is multi-model. In short:

- Single-vendor hosts resolve by themselves — Claude Code → Anthropic, Codex → OpenAI, Gemini CLI / Antigravity / Jules → Google, Grok Build → xAI.
- Every other host in the table above is multi-model or user-configured, and routes to the universal baseline unless the selected model is visible.

When you add an adapter row above, add the host to that map too, or the skill will silently write baseline prompts on a host whose vendor is actually known.

## Adapter rule

Keep adapters thin. When a host supports skills or commands, point it at the
existing `skills/` and `commands/` files rather than duplicating them. When a
host only supports project instructions, its rule file is a byte-copy of the
canonical body in `AGENTS.md` (everything above the `<!-- shared-rule-end -->`
marker), plus host-specific frontmatter. Fixture G1 in
`tests/activation-fixtures.md` fails on drift.

Regenerate every instruction-tier copy from the canonical body:

```bash
sed '/<!-- shared-rule-end -->/,$d' AGENTS.md | sed -e :a -e '/^\n*$/{$d;N;ba' -e '}' > /tmp/pbp-body.md
for f in .windsurf/rules/prompt-best-practices.md .clinerules/prompt-best-practices.md \
         rules/prompt-best-practices.md .qoder/rules/prompt-best-practices.md \
         .github/copilot-instructions.md; do cp /tmp/pbp-body.md "$f"; done
```

`.cursor/rules/prompt-best-practices.mdc` and `.kiro/steering/prompt-best-practices.md`
carry frontmatter above the same body — replace the body below their `---` block.

## Not shipped, and why

- **Hook scripts and persistent state**: the hooks that do ship are a single `echo` per host. There is no mode to track, no flag file, no statusline, and no `node` dependency — the skill has no persistent state, only an activation reminder. (Until 0.8.0 no hook shipped at all, on the reasoning that there was nothing to inject each turn. That was wrong about one thing: the activation rule itself needs to be in context before the agent decides, which is what 0.9.0 fixed.)
- **MCP server, OpenCode server plugin, pi extension code**: they exist to inject instructions each turn. On those hosts `AGENTS.md` is already auto-loaded, so the injection is redundant.
- **Hermes Agent plugin** (`plugin.yaml` + `__init__.py`): its plugin format expects a Python entry point registering hooks and commands. Use the instruction-tier path (`AGENTS.md`) or copy `skills/` manually.
- **Flattened `.openclaw/skills/` copy**: the skill is a folder with `references/`, so a flat `SKILL.md` copy would lose guidance and duplicate 12 files. OpenClaw reads `skills/prompt-best-practices/` directly.

## Portable behavior

- `skills/prompt-best-practices/SKILL.md` — full workflow: activation rules, two-pass assessment, tier-capped dialogue, final prompt build, per-model gates.
- `skills/prompt-best-practices/references/` — framework, component definitions, rubrics, examples, per-model tuning notes.
- `AGENTS.md` (above the marker) — compact always-on rule for hosts without skill support.
- `commands/prompt-best-practices.toml`, `.opencode/commands/prompt-best-practices.md` — self-contained command prompt for hosts that only read commands.
