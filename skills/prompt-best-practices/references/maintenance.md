# Framework Maintenance and Update Procedure

## When to Use

**Contributor-facing file. Do not load it to build a prompt.** Load it only when verifying the skill against updated official documentation, adding a newly released model, or preparing a release.

It holds three things: the verification record (what was checked, when, against which models), the provenance of the secondary reference files, and the step-by-step update procedure.

For runtime work, use `framework.md` (tiers, calibration, template) and `component-definitions.md` (component definitions).

---

## Verification record

| Field | Value |
|---|---|
| Source | [Claude prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) |
| Last verified | 2026-08-03 |
| Verified against | Claude Opus 5 (`claude-opus-5`) — current default target; Claude Fable 5 (`claude-fable-5`) and Claude Mythos 5 — highest-capability tier; Claude Sonnet 5; Claude Opus 4.8 (`claude-opus-4-8`) — previous generation and recommended refusal-fallback target. Cross-checked against OpenAI GPT-5.6 Sol guidance. |
| Source fingerprint | Page organized in three parts (model-specific guidance / techniques for all current models / migration). Model-specific guidance lives on **four** dedicated per-model pages (`prompting-claude-opus-5`, `prompting-claude-fable-5`, `prompting-claude-sonnet-5`, `prompting-claude-opus-4-8`), not in-page anchors. General-principles anchors stable: `be-clear-and-direct`, `give-claude-a-role`, `long-context-prompting`, `use-examples-effectively`, `control-the-format-of-responses`, `add-context-to-improve-performance`, `structure-prompts-with-xml-tags`. Two general-technique sections now carry an explicit **Claude Opus 5 exception**: `leverage-thinking--interleaved-thinking-capabilities` (self-check → remove the instruction on Opus 5) and `communication-style-and-verbosity` (effort does not shorten visible output; prompt for conciseness). The `use-examples-effectively` quote dropped "dramatically" ("examples … improve accuracy and consistency"). The "refactor prompts and skills / do not over-prescribe" directive is shared by Anthropic (Fable 5) and OpenAI (GPT-5.6 lean prompts) — captured in `framework.md` > "Calibrating for frontier models". Recompute on update. |

The fingerprint is a short checklist of the named sections and key quotes the framework depends on, so a contributor can tell at a glance whether the source page shifted meaningfully since the last verification.

---

## Additional source mapping

Beyond the seven per-component sources listed in `framework.md` > Source mapping, the files `grounding-techniques.md`, `claude-considerations.md`, and the per-model files (`claude-opus-5.md`, `claude-fable-5.md`) derive from:

- [Thinking and reasoning > Overthinking and excessive thoroughness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#overthinking-and-excessive-thoroughness)
- [Agentic systems > Overeagerness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#overeagerness)
- [Tool use > Tool usage](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#tool-usage)
- [Agentic systems > Minimizing hallucinations in agentic coding](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#minimizing-hallucinations-in-agentic-coding)
- [General principles > Long context prompting](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#long-context-prompting) (grounding in quotes)
- [Thinking and reasoning > Leverage thinking](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#leverage-thinking--interleaved-thinking-capabilities) (self-check)

---

## Update procedure

Follow this procedure when the official guide is updated.

1. **Fetch the latest version** of the source page:
   `https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices`

2. **Recompute the source fingerprint.** Compare against the `Source fingerprint` row above. If any fingerprint item is missing or materially changed, the framework needs patching.

3. **Check each component against its source section.** Use `framework.md` > Source mapping to verify each component is still aligned, and `component-definitions.md` for the full definition. For each row:
   - Does the source section still exist at the linked anchor?
   - Has the key quote changed or been removed?
   - Are there new recommendations that should be reflected?

4. **Check for new sections.** Scan the guide for sections not covered by any existing component. If a new principle is introduced, evaluate whether it warrants a new component or an update to an existing one.

5. **Check the Claude-specific considerations.** Model-specific guidance lives on dedicated `prompting-claude-<model>` pages (`prompting-claude-opus-5`, `prompting-claude-fable-5`, `prompting-claude-sonnet-5`, `prompting-claude-opus-4-8`), not in-page anchors, and this repo mirrors that layout: `claude-considerations.md` is the router, one `claude-<model>.md` file per tracked model. When a new model ships, follow the update procedure in `claude-considerations.md` — in particular, re-diff the "Pick the model before you tune" table, since new generations tend to invert prior tuning rather than extend it.

6. **Re-run the activation fixtures.** `tests/activation-fixtures.md` lists reference prompts with expected activation outcomes. Any edit to `SKILL.md`, `framework.md`, `component-definitions.md`, or `component-rubrics.md` must leave all fixtures producing the expected outcome.

7. **Update the verification record** above with the new date, model generation, and recomputed fingerprint.

8. **Update `SKILL.md`** if any component was added, removed, or renamed — the component list in Step 0, the tier mapping, the Reference Map, and the dialogue priorities in Step 2 must match `framework.md` and `component-definitions.md`.

9. **Update `component-rubrics.md`** if the definition of any component changed — the operational checklist has to reflect the new definition.

10. **Update `examples-content.md` and `examples-code.md`** if the scoring denominator changed (e.g., `/7` → `/8`) or if example final prompts need to reflect new template structure.

11. **Re-check the file budgets.** No reference file may exceed 4000 tokens (~2700 words) and `SKILL.md` may not exceed 5000 tokens (~3500 words) — a file over budget risks being truncated or skipped by the loading agent. Measure with:
    ```sh
    wc -w skills/prompt-best-practices/SKILL.md skills/prompt-best-practices/references/*.md
    ```
    If a file is over, split it by topic and add the new file to the Reference Map in both `SKILL.md` and `framework.md`.

12. **Update the version** in `.claude-plugin/plugin.json` (single source of truth) and add a new entry in `CHANGELOG.md`.

---

## Important note

This framework is a practical tool, not a rigid prescription. The primary source for the principles behind it is [Anthropic's prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). The best prompting approach always depends on the specific task — use the components that add value and skip those that do not. A well-written 2-component prompt will always outperform a poorly thought-out 7-component one.
