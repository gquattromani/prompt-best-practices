# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-04-15

### Added

- Interactive prompt structuring skill using a 7-component framework derived from Anthropic's official prompting best practices. Each component maps to a specific section of the documentation.
- 4-step workflow: Quick Assessment, Diagnosis, Guided Dialogue (one question at a time), Build Final Prompt. Optional Step 4 for refinement.
- Fast path for experienced users: skip dialogue for components already provided.
- Language detection: responds in the same language as the user's prompt.
- Model-agnostic language throughout instructional text. Claude-specific guidance isolated in `references/claude-considerations.md`.
- 6 dialogue examples (3 complete with final prompt, 2 abbreviated, 1 no-activation) and 3 anti-pattern examples.
- Grounding and accuracy section with 3 techniques for hallucination prevention.
- Plugin manifest (`.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`) and Agent Skills standard support (`AGENTS.md`).
- Reference files: `framework.md` (component definitions, source mapping, prompt template, update procedure), `examples.md` (dialogue examples), `claude-considerations.md` (Claude 4.6 specific tips).
