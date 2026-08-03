# Agent Instructions

This repository contains an Agent Skill following the
[Agent Skills open standard](https://agentskills.io/).

## Available Skills

| Skill | Location | Description |
|---|---|---|
| prompt-best-practices | `skills/prompt-best-practices/SKILL.md` | Interactive prompt structuring using a 7-component framework derived from Anthropic's official prompting best practices |

## Usage

The `skills/prompt-best-practices/` folder contains a `SKILL.md` entry point. Load and follow the
instructions in `SKILL.md` when working on matching tasks.

Reference files in `skills/prompt-best-practices/references/` provide detailed topic guidance. Each one is self-contained, opens with a `## When to Use` section, and stays under 4000 tokens so it can be loaded in full. `SKILL.md` > Reference Map is the authoritative "load this when" table.

- `framework.md` — **entry point**: task tiers, the 7 components at a glance, the "Calibrating for frontier models" guidance, the final prompt template
- `component-definitions.md` — canonical definition of each of the 7 components (format, what to check for, present/missing criteria)
- `component-rubrics.md` — operational checklist for classifying each component as `[OK]` / `[~~]` / `[--]`
- `examples-content.md` / `examples-code.md` — dialogue examples (content tasks / code tasks)
- `grounding-techniques.md` — techniques to prevent hallucinations (technique 3, self-check, is model-gated)
- `claude-considerations.md` — Anthropic Claude router: per-model file selection, family-wide behaviors, and the divergence table where per-model tuning conflicts
- `claude-opus-5.md` — tuning notes for Claude Opus 5 (default target)
- `claude-fable-5.md` — tuning notes for Claude Fable 5 / Mythos 5 (highest-capability tier)
- `codex-considerations.md` — tuning notes for OpenAI Codex (GPT-5.6 Sol flagship, Terra / Luna siblings)
- `maintenance.md` — **contributor-only**: verification record, source fingerprint, update procedure. Do not load it to build a prompt.

Test assets in `tests/`:

- `activation-fixtures.md` — reference prompts with expected activation outcomes. Run before merging any change to `SKILL.md`, `framework.md`, `component-definitions.md`, or `component-rubrics.md`. Fixture set F is three shell commands that verify token budgets, the `## When to Use` headers, and that every path named in `SKILL.md` resolves.
- `benchmark-protocol.md` — protocol for measuring the framework's effect on output quality.
