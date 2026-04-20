# Agent Instructions

This repository contains an Agent Skill following the
[Agent Skills open standard](https://agentskills.io/).

## Available Skills

| Skill | Location | Description |
|---|---|---|
| prompt-best-practices | `skill/SKILL.md` | Interactive prompt structuring using a 7-component framework derived from Anthropic's official prompting best practices |

## Usage

The `skill/` folder contains a `SKILL.md` entry point. Load and follow the instructions
in `SKILL.md` when working on matching tasks.

Reference files in `skill/references/` provide detailed topic guidance:

- `framework.md` — canonical definitions for the 7 components, the task-tier mapping, and the final prompt template
- `component-rubrics.md` — operational checklist for classifying each component as `[OK]` / `[~~]` / `[--]`
- `examples-content.md` / `examples-code.md` — dialogue examples (content tasks / code tasks)
- `grounding-techniques.md` — techniques to prevent hallucinations
- `claude-considerations.md` — tuning notes when the target model is Anthropic Claude
- `codex-considerations.md` — tuning notes when the target model is OpenAI Codex

Test assets in `tests/`:

- `activation-fixtures.md` — reference prompts with expected activation outcomes. Run before merging any change to `SKILL.md`, `framework.md`, or `component-rubrics.md`.
- `benchmark-protocol.md` — protocol for measuring the framework's effect on output quality.
