# Framework Maintenance and Update Procedure

## When to Use

**Contributor-facing file. Do not load it to build a prompt.** Load it only when verifying the skill against updated official documentation, adding a newly released model, or preparing a release.

It holds three things: the verification record (what was checked, when, against which models), the provenance of the secondary reference files, and the step-by-step update procedure.

For runtime work, use `framework.md` (tiers, calibration, template), `component-definitions.md` (component definitions), and `runtime-detection.md` (host-to-vendor routing).

---

## Verification record

### Anthropic (primary source)

| Field | Value |
|---|---|
| Source | [Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| Last verified | 2026-09-25 |
| Verified against | Claude Opus 5.5 (`claude-opus-5-5`) — current default target, and the default model in Claude Code on paid plans and the API; Claude Fable 5.1 / Mythos 5.1 — highest-capability tier; Claude Opus 5, Fable 5 / Mythos 5, Sonnet 5, Opus 4.8. Cross-checked against OpenAI GPT-6 guidance. |
| Source fingerprint | Three-part layout unchanged (model-specific guidance / techniques for all current models / migration). Model-specific guidance lives on **six** per-model pages (`prompting-claude-fable-5-1`, `prompting-claude-fable-5`, `prompting-claude-sonnet-5`, `prompting-claude-opus-5-5`, `prompting-claude-opus-5`, `prompting-claude-opus-4-8`), each written as a delta from its predecessor. General-principles anchors stable: `be-clear-and-direct`, `give-claude-a-role`, `long-context-prompting`, `use-examples-effectively`, `control-the-format-of-responses`, `add-context-to-improve-performance`, `structure-prompts-with-xml-tags`. New since 2026-08-03: the general-principles intro says a model-named technique is "measured on that model"; `communication-style-and-verbosity` and `control-the-format-of-responses` each gained a **Fable 5.1 exception** (fewer progress updates; remove anti-markdown blocks); `leverage-thinking--interleaved-thinking-capabilities` lists Fable 5.x and Opus 5.5 as thinking-always-on and its self-check sentence now reads "On Claude Opus 5, remove these instructions rather than rewriting them"; `long-context-prompting` carries the "up to 30 percent" figure again, in a Note; migration item 7 (append-only history on Fable 5.1 and Opus 5.5) is new. `use-examples-effectively` still says "Include 3–5 examples for best results". Recompute on update. |

The fingerprint lists the sections and quotes the framework depends on, so a shifted page shows at a glance.

**Quotation check, 2026-09-25.** Every page was downloaded as Markdown (append `.md` to Anthropic and OpenAI URLs, `.md.txt` to Google's) and every double-quoted passage in the reference files was checked against the downloads mechanically. Corrections: the self-check quotation in `grounding-techniques.md` had drifted; four Gemini quotations cited the deprecated Gemini 3 guide and were moved to the current pages; two paraphrases in `universal-baseline.md` and one in `component-definitions.md` sat in quotation marks and are now verbatim; the OpenAI model-table quotations and the xAI recommendation were rewritten with the GPT-6 and Grok 4.7 pages. **Not re-verifiable:** the four quotations from xAI's `grok-code-fast-1` prompt-engineering guide — the guide is no longer in the xAI docs index and its known URLs return 404 — so they stand on the 2026-08-19 check (`grok-considerations.md` > "Known gap").

**Source terms and quotation policy.** Google's pages carry the footer "Except as otherwise noted, the content of this page is licensed under the Creative Commons Attribution 4.0 License" (re-confirmed 2026-09-25 on all three pages cited), so the CC BY basis in `NOTICE.md` holds. Anthropic, OpenAI, and xAI pages carry no licence notice: the basis stays the right of quotation, and no vendor name appears in this project's product descriptions. Verbatim material is limited to the passages in double quotes, each attributed and linked; the instruction blocks in `claude-opus-5-5.md` and `claude-fable-5.md` are condensed adaptations in this project's own words. When a page updates, re-derive them rather than pasting the vendor's sample prompts back in, and keep each quotation to the sentence that carries the point (basis: `NOTICE.md`). xAI's documentation now brands the vendor "SpaceXAI"; the skill keeps "xAI", the form the docs still give in parentheses.

### Other vendors

Each vendor file carries its own `Sources` block and update procedure; this table is the release-level record of when the branch was last checked.

| Vendor | Files | Pages verified | Last verified | Fingerprint |
|---|---|---|---|---|
| OpenAI | `codex-considerations.md` | [Using GPT-6](https://developers.openai.com/api/docs/guides/latest-model/gpt-6-astra), [GPT-5.6 Sol guidance](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6), [Reasoning](https://developers.openai.com/api/docs/guides/reasoning), [models](https://learn.chatgpt.com/docs/models), [prompting](https://learn.chatgpt.com/docs/prompting) | 2026-09-25 | GPT-6 family (Astra, Sol, Luna); GPT-6 behavior list: asks more, sensitive to skills and `AGENTS.md`, formatted by default, delegates less, tests thoroughly; lean-prompt figures still on the GPT-5.6 page ("roughly 10–15%", "41–66%", "33–67%"); `reasoning.effort` `none`…`max`, `medium` default on Sol and Luna, no `none` on Astra; sampling parameters removed when reasoning is on; `apply_patch` tool with V4A diffs; GPT-5.5 retires from ChatGPT and Codex on 2026-10-14. |
| Google | `gemini-considerations.md` | [Prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies) (updated 2026-09-17), [Gemini 3.8 Flash](https://ai.google.dev/gemini-api/docs/latest-model) and [Gemini 3.5 Flash](https://ai.google.dev/gemini-api/docs/whats-new-gemini-3.5) (both updated 2026-09-23) | 2026-09-25 | Gemini 3.8 Flash GA and the Antigravity agent default; the Gemini 3 guide is deprecated; "Be concise. Gemini 3.x responds best to direct, clear instructions."; terse default; instructions last after data; XML tags **or** Markdown headings; sampling parameters "Remove these parameters from all requests."; `thinking_level` default `medium` on 3.5 and 3.8 Flash, `high` on 3.1 Pro, no `minimal` on 3.8; "Think very hard before answering" still endorsed; 3.8 Flash verifies its own work; "always include few-shot examples" unchanged. |
| xAI | `grok-considerations.md` | [Grok 4.7](https://docs.x.ai/developers/grok-4-7), [Models](https://docs.x.ai/developers/models), [Reasoning](https://docs.x.ai/developers/model-capabilities/text/reasoning), [Grok Build overview](https://docs.x.ai/build/overview) | 2026-09-25 | `grok-4.7` flagship and Grok Build default; `grok-build-0.1` is the coding model (`grok-code-fast-1` alias); `reasoning_effort` `low`/`medium`/`high` (default)/`xhigh`, cannot be disabled; `prompt_cache_key` "highly recommend" unchanged. **Known gap:** the composition guidance is a transfer from the coding-model guide, which could not be reopened this pass. |
| Host map | `runtime-detection.md` | Each host's own docs, plus [GitHub Copilot supported models](https://docs.github.com/en/copilot/reference/ai-models/supported-models) | 2026-09-25 | Claude Code defaults to Opus 5.5; Codex recommends GPT-6 Sol; Gemini CLI replaced by Antigravity CLI for unpaid-tier and Google One users on 2026-06-18; Antigravity picker lists Gemini 3.x plus Claude and GPT-OSS models; Grok Build defaults to `grok-4.7` and accepts custom models; Copilot vendors unchanged (Anthropic, OpenAI, Google, xAI, Microsoft, Moonshot AI); Windsurf renamed Devin Desktop; every other host-selected row still multi-model. |

The host map is the most perishable row: hosts add vendors and change defaults between releases, so re-verify it on every release even when no vendor page moved.

---

## Additional source mapping

`gemini-considerations.md`, `grok-considerations.md`, and the host map in `runtime-detection.md` derive from the vendor pages listed in the "Other vendors" table above; `universal-baseline.md` derives from the intersection of all four vendors and cites no source of its own — every rule in it must be traceable to at least one vendor file.

Beyond the seven per-component sources listed in `framework.md` > Source mapping, the files `grounding-techniques.md`, `claude-considerations.md`, and the per-model files (`claude-opus-5-5.md`, `claude-fable-5.md`) derive from:

- [Thinking and reasoning > Overthinking and excessive thoroughness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#overthinking-and-excessive-thoroughness)
- [Agentic systems > Overeagerness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#overeagerness)
- [Tool use > Tool usage](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#tool-usage)
- [Agentic systems > Minimizing hallucinations in agentic coding](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#minimizing-hallucinations-in-agentic-coding)
- [General principles > Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting) (grounding in quotes)
- [Thinking and reasoning > Leverage thinking](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#leverage-thinking--interleaved-thinking-capabilities) (self-check)

## Host adapter verification record

Every adapter this repository ships, checked against that host's own documentation on
2026-08-19. A row saying **unconfirmed** means the adapter file exists but the host's
documentation does not describe the format it assumes — it is not a claim that the
adapter fails, it is a claim that nothing here proves it works.

Not re-run for 0.10.0: no adapter was added. Two host facts moved since and are unverified
against the adapters: Windsurf is now Devin Desktop, so whether it still reads
`.windsurf/rules/` needs checking; pi moved to the `earendil-works/pi` repository.

| Host | Documentation read | Verdict |
|---|---|---|
| Claude Code | verified by use, not by doc: `/plugin marketplace add` + `/plugin install` both succeeded on 2026-08-19 | **Confirmed.** The host accepted `.claude-plugin/marketplace.json` and `plugin.json`. |
| Codex | [Build skills](https://learn.chatgpt.com/docs/build-skills) | **Unconfirmed.** Skills are documented as loading from `.agents/skills` (cwd, parents, repo root), `$HOME/.agents/skills`, `/etc/codex/skills`. No `.codex-plugin/plugin.json` and no `commands/*.toml` auto-discovery appear in the doc; the documented per-skill metadata file is `agents/openai.yaml`. |
| GitHub Copilot CLI | [CLI plugin reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference), [Creating a plugin](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating), [Creating a marketplace](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-marketplace) | **Confirmed.** `plugin.json` at the plugin root; `skills` and `commands` accept `string | string[]`; `hooks` accepts a path string; `marketplace.json` belongs in `.github/plugin/` with `name`, `owner`, `plugins`. |
| GitHub Copilot (skills discovery) | [About Agent Skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) | **Noted.** Copilot discovers skills from `.github/skills`, `.claude/skills`, `.agents/skills`, `~/.copilot/skills`, `~/.agents/skills`. A repository-root `skills/` reaches Copilot only through the plugin manifest, never by discovery. |
| Gemini CLI | [Extension manifest reference](https://geminicli.com/docs/extensions/reference/), [Skills](https://geminicli.com/docs/cli/skills/) | **Confirmed.** `gemini-extension.json` must sit in the extension root; `contextFileName` is a documented key; `commands/` TOML files and a `skills/` directory inside an extension are both auto-discovered. |
| Antigravity CLI (`agy`) | [Plugins & Skills](https://antigravity.google/docs/cli/plugins) | **Confirmed after a rebuild.** The documented manifest is `plugin.json` at the plugin root, `name` required and `description` optional; the documented layout is `plugin.json`, `mcp_config.json`, `hooks.json`, `skills/`, `agents/`, `rules/`. The adapter previously relied on `gemini-extension.json` and a `.agents/plugins/marketplace.json`, neither documented, and kept its rule copy in the undocumented `.agents/rules/`; the marketplace file was removed and the rule copy moved to `rules/`. Install is documented only as `agy plugin install /path/to/local/plugin`. |
| Grok Build | [Skills, Plugins & Marketplaces](https://docs.x.ai/build/features/skills-plugins-marketplaces) | **Unconfirmed.** Plugins load from `./.grok/plugins/`, `~/.grok/plugins/` and marketplace installs under `~/.grok/plugins/marketplaces/`; skills come from an enabled plugin's `skills/` directory. The manifest filename and any repository-side marketplace path are not documented on that page. |
| Qoder | [Plugins](https://docs.qoder.com/cli/plugins), [Rules](https://docs.qoder.com/user-guide/rules) | **Confirmed after two fixes.** `.qoder-plugin/plugin.json` is the documented location and `.qoder/rules/` plus `AGENTS.md` are both documented. The manifest previously declared a `rules` key, which is not in the documented field set, and an object `author`, which is documented as a string; both corrected. |
| OpenCode | [Agent Skills](https://opencode.ai/docs/skills/), [Commands](https://opencode.ai/docs/commands/) | **Confirmed after a fix.** Commands live in `.opencode/commands/` — plural — and the directory shipped here was `command/`; renamed. Skills are documented under `.opencode/skills/`, `.claude/skills/`, `.agents/skills/` and their global equivalents, so a repository-root `skills/` is not a documented discovery path. |
| pi | [pi skills](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/skills.md) (repository now at `earendil-works/pi`) | **Partly confirmed.** `pi.skills` entries in `package.json` and a package's own `skills/` directory are both documented as registration mechanisms. The exact shape of the field and a git install command are not documented. |
| OpenClaw | [Skills](https://docs.openclaw.ai/tools/skills) | **Confirmed.** `<workspace>/skills` is a documented discovery root, folders nest freely, and `openclaw skills install git:owner/repo@ref` plus `--global` are documented. |
| Swival | not located | **Unverified.** No first-party documentation was found for the install command or the instruction-file paths this repository claims. |
| Cursor | [Rules](https://cursor.com/docs/context/rules), [Skills](https://cursor.com/docs/context/skills) | **Confirmed.** `.cursor/rules/*.mdc` with `alwaysApply`, `description`, `globs`. The `.mdc` extension is load-bearing: a plain `.md` in that directory is ignored. Skills, separately, load from `.agents/skills/` and `.cursor/skills/`. |
| Windsurf | [Creating & modifying rules](https://windsurf.com/university/general-education/creating-modifying-rules) | **Confirmed, with a budget caveat.** Workspace rules live under `.windsurf/`. Global and workspace rules share a **12,000-character** budget; the rule copy here is roughly 5,600-5,900 characters, so it consumes about half of a user's total allowance. |
| Cline | [Cline rules](https://docs.cline.bot/features/cline-rules) | **Confirmed.** `.clinerules/` is read as a directory of `.md` files and frontmatter is optional. |
| Kiro | [Steering](https://kiro.dev/docs/steering/) | **Confirmed.** `.kiro/steering/*.md`, `inclusion: always` is the default value, and `~/.kiro/steering/` works globally. |
| Zed | [Instructions](https://zed.dev/docs/ai/instructions), [Rules](https://zed.dev/docs/ai/rules) | **Confirmed.** `AGENTS.md` is supported for always-on project instructions. |
| Amp | [Amp manual](https://ampcode.com/manual) | **Confirmed.** `AGENTS.md` from the working directory up to `$HOME`, plus `$HOME/.config/amp/AGENTS.md`, with `AGENT.md` and `CLAUDE.md` as fallbacks. |
| JetBrains Junie | [Agent Skills](https://junie.jetbrains.com/docs/agent-skills.html) | **Unconfirmed.** Junie's documented skill paths are `.junie/skills/` and `~/.junie/skills/`, neither of which ships here. The claim that a guidelines path can be pointed at `AGENTS.md` lives on a separate page that was not read. |
| Aider | [Conventions](https://aider.chat/docs/usage/conventions.html), [YAML config](https://aider.chat/docs/config/aider_conf.html) | **Confirmed.** `--read FILE` on the command line and `read:` in `.aider.conf.yml` are both documented. |
| Jules, CodeWhale, VS Code + Codex extension | not located | **Unverified.** These rows assert that a root instruction file is read; no first-party page was found for them in this pass. |
| Agent Skills format | [Specification](https://agentskills.io/specification) | **Confirmed by mechanical check.** `name` matches the parent directory and is within 64 characters; `description` is within 1024; the body is well under the 500-line guidance; references are one level deep. |

**What this record does not cover.** Every *unconfirmed* row above is an adapter whose
format could not be traced to first-party documentation. That is not a claim that the
adapter fails, only that nothing here proves it works. Two rows were acted on rather than
labelled: the **Devin** adapter was removed outright, because the format it assumed appears
nowhere in Devin's documentation, and the **Antigravity** adapter was rebuilt to the
documented shape. Re-run this record whenever an adapter is added, and never add a row for
a host whose manifest documentation has not been opened.

---

## Update procedure

Follow this procedure when the official guide is updated.

1. **Fetch the latest version** of the source page, as Markdown so quotations can be checked mechanically:
   `https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices.md`

2. **Recompute the source fingerprint.** Compare against the `Source fingerprint` row above. If any fingerprint item is missing or materially changed, the framework needs patching.

3. **Check each component against its source section.** Use `framework.md` > Source mapping to verify each component is still aligned, and `component-definitions.md` for the full definition. For each row:
   - Does the source section still exist at the linked anchor?
   - Has the key quote changed or been removed?
   - Are there new recommendations that should be reflected?

4. **Check for new sections.** Scan the guide for sections not covered by any existing component. If a new principle is introduced, evaluate whether it warrants a new component or an update to an existing one.

5. **Check every vendor branch, not just Anthropic.** Follow the update procedure at the bottom of `gemini-considerations.md` and `grok-considerations.md`, then re-verify the host map in `runtime-detection.md` — a host that switched default vendor silently misroutes every prompt built in it. When a vendor's guidance changes, re-diff the cross-vendor divergence table in `runtime-detection.md` and re-check whether any rule in `universal-baseline.md` has stopped being unanimous: a rule that one vendor now contradicts must move out of the baseline into that vendor's file.

6. **Check the Claude-specific considerations.** Model-specific guidance lives on dedicated `prompting-claude-<model>` pages (six as of 2026-09-25 — see the fingerprint above), not in-page anchors, and this repo mirrors that layout: `claude-considerations.md` is the router, one `claude-<model>.md` file per tuning lineage (`claude-opus-5-5.md` covers Opus 5 and 5.5; `claude-fable-5.md` covers Fable 5 and 5.1). When a new model ships, follow the update procedure in `claude-considerations.md` — in particular, re-diff the "Pick the model before you tune" table, since new generations tend to invert prior tuning rather than extend it.

7. **Re-run the activation fixtures.** `tests/activation-fixtures.md` lists reference prompts with expected activation outcomes. Any edit to `SKILL.md`, `framework.md`, `component-definitions.md`, or `component-rubrics.md` must leave all fixtures producing the expected outcome.

8. **Update the verification record** above with the new date, model generation, and recomputed fingerprint.

9. **Update `SKILL.md`** if any component was added, removed, or renamed — the component list in Step 0, the tier mapping, the Reference Map, and the dialogue priorities in Step 2 must match `framework.md` and `component-definitions.md`.

10. **Update `component-rubrics.md`** if the definition of any component changed — the operational checklist has to reflect the new definition.

11. **Update `examples-content.md` and `examples-code.md`** if the scoring denominator changed (e.g., `/7` → `/8`) or if example final prompts need to reflect new template structure.

12. **Re-check the file budgets.** No reference file may exceed 4000 tokens (~2700 words) and `SKILL.md` may not exceed 5000 tokens (~3500 words) — a file over budget risks being truncated or skipped by the loading agent. Measure with:
    ```sh
    wc -w skills/prompt-best-practices/SKILL.md skills/prompt-best-practices/references/*.md
    ```
    If a file is over, split it by topic and add the new file to the Reference Map in both `SKILL.md` and `framework.md`.

13. **Update the version** in `.claude-plugin/plugin.json` (single source of truth) and add a new entry in `CHANGELOG.md`.

---

## Important note

This framework is a practical tool, not a rigid prescription. The primary source for the principles behind it is [Anthropic's prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). The best prompting approach always depends on the specific task — use the components that add value and skip those that do not. A well-written 2-component prompt will always outperform a poorly thought-out 7-component one.
