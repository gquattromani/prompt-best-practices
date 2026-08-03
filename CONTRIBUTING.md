# Contributing

Thanks for your interest in improving this skill!

This repository follows the [Agent Skills open standard](https://agentskills.io/).

## Scope and non-goals

This project is intentionally small. Changes that expand the surface area rarely help; changes that sharpen the existing surface almost always do. Before opening a PR, confirm the change fits one of the in-scope categories below.

**In scope:**
- Improvements to the 7-component framework that track the [official Anthropic guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices).
- Clarity fixes in `SKILL.md`, the reference files, and examples.
- New rubrics, tier signals, or activation fixtures that make behavior more deterministic.
- Benchmark results from running `tests/benchmark-protocol.md`.
- Model-specific tuning notes when a new generation ships.

**Out of scope:**
- Features that turn the skill into a broader agent (scheduling, tool orchestration, code execution, UI).
- New prompt components beyond the 7 in the framework, unless the Anthropic guide introduces one.
- Project-management plumbing (issue templates, CI bots, bots that rewrite Markdown) that is not tied to a concrete quality improvement in the skill itself.
- Alternative frameworks to the 7-component one. If a different framework is better, fork — don't graft.

When in doubt, prefer depth over breadth. Making the existing components more reliable beats adding a new one.

## What You Can Contribute

- **Improve SKILL.md** — clearer instructions, better examples, fewer ambiguities
- **Add or improve references** — deeper guidance on specific topics
- **Add activation fixtures** — reference prompts with expected outcomes that protect the skill from regressions (see `tests/activation-fixtures.md`)
- **Run the benchmark** — empirical evidence for the framework's effect size (see `tests/benchmark-protocol.md`)
- **Fix typos and formatting**

## Guidelines

### SKILL.md (the entry point)
- Keep the body under **5000 tokens** (~3500 words)
- Write for AI agents, not humans — be direct, unambiguous, actionable
- Don't repeat what's in reference files — point to them instead
- Always write reference paths with the `references/` prefix. A bare `framework.md` resolves against the agent's working directory and fails to open.
- Every reference file must appear in the Reference Map table, and in its sibling table in `framework.md` > When to Use. Keep the two in sync — a file nobody is told to load is dead content.

### Reference files (references/)
- One topic per file, hard ceiling **4000 tokens** (~2700 words) — no exceptions, including the canonical files. A file over budget risks being truncated or skipped by the loading agent, which is indistinguishable from the guidance not existing. If a file grows past the ceiling, split it by topic rather than trimming substance.
- Each file must be self-contained — agents load them independently
- Use concrete examples, not abstract explanations
- Include a `## When to Use` section at the top of every file — it tells an agent whether loading the file is worth the tokens
- Cross-reference by section title (`` `framework.md` > "Calibrating for frontier models" ``), not by line number. When you move a section between files, update every inbound reference in the same commit.

### Budget check

Run this from the repository root before opening a PR:

```sh
wc -w skills/prompt-best-practices/SKILL.md skills/prompt-best-practices/references/*.md
```

`SKILL.md` must stay ≤ 3500 words and every reference file ≤ 2700 words. This is fixture F1 in `tests/activation-fixtures.md`; F2 and F3 in the same set check the `## When to Use` header and the resolvability of every path named in `SKILL.md`.

## Pull Request Process

1. Fork the repository
2. Create a descriptive branch name
3. Make your changes
4. Test with at least one AI agent (Claude Code, Codex, Cursor, etc.)
5. **Run the activation fixtures** in `tests/activation-fixtures.md` — any change to `SKILL.md`, `framework.md`, `component-definitions.md`, or `component-rubrics.md` must leave every fixture producing its expected outcome. Fixture set F (loadability) is three shell commands; run it on any change that adds, splits, renames, or grows a file.
6. If the change is likely to affect output quality, run `tests/benchmark-protocol.md` and include the results file in your PR.
7. Submit a PR describing what changed and why

## Code of Conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). By participating, you agree to uphold it.

## Security

Found a vulnerability or a prompt pattern that bypasses the skill's intended behavior? See [SECURITY.md](SECURITY.md) for private reporting instructions — please do not open a public issue.
