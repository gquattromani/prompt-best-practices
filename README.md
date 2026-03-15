# Prompt Best Practices

An AI agent skill that transforms unstructured prompts into high-quality structured prompts through a short interactive dialogue. Works with any AI agent that supports the [Agent Skills](https://agentskills.io/) open standard.

The 7-component framework was built by studying the prompting guidelines published by the major LLM providers. The primary reference is [Anthropic's prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices), the most comprehensive publicly available guide on the subject, with each component mapped to a specific section of that documentation. The underlying principles — clear tasks, structured context, concrete examples, well-motivated constraints — are shared across providers and improve output quality on any LLM.

## The Problem

Most prompts sent to AI agents lack the structure needed to produce reliable, high-quality output. Users provide a vague goal — "write me an email", "build this feature" — without defining success criteria, output format, constraints, or context. The model fills in the blanks with assumptions, and what follows is multiple rounds of corrections to converge on what the user actually wanted.

This skill eliminates that cycle. It intercepts underspecified prompts and guides you through 3-5 targeted questions to surface the missing information. The result is a structured prompt that any LLM-based agent can execute correctly on the first pass.

## Demo

<p align="center">
  <img src="assets/demo.gif" alt="Prompt Best Practices skill demo" width="800">
</p>

## Installation

One command. No configuration, no dependencies, no setup files to edit.

### Claude Code Plugin (recommended)

```
/plugin marketplace add gquattromani/prompt-best-practices
/plugin install prompt-best-practices@prompt-best-practices
```

### Via skills.sh (any compatible agent)

```bash
npx skills add gquattromani/prompt-best-practices -g -y
```

This installs the skill globally. It is immediately available in every project, in every agent that supports the [Agent Skills](https://agentskills.io/) standard.

## Usage

```
/prompt-best-practices
```

The skill activates automatically when you ask to write, draft, generate, implement, fix, refactor, or build any output that lacks clear structure. Invoke it explicitly via slash command to force activation on any prompt.

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

If fewer than 3 components are present, the skill starts a guided dialogue to fill the gaps. Once complete, it assembles a structured prompt and executes it.

## Compatible Platforms

This skill follows the [Agent Skills](https://agentskills.io/) open standard. It works out of the box with:

Claude Code, Codex, Cursor, GitHub Copilot, Windsurf, Roo Code, Goose, Gemini CLI, [and more](https://agentskills.io/).

Any agent that reads `AGENTS.md` and follows Markdown-based skill definitions can use it without modification.

## Uninstall

### Via CLI

```bash
claude plugin uninstall prompt-best-practices
```

To specify the scope:

```bash
claude plugin uninstall prompt-best-practices --scope user      # global (default)
claude plugin uninstall prompt-best-practices --scope project    # project-level
claude plugin uninstall prompt-best-practices --scope local      # local only
```

### Manual removal

Remove the skill entry from the relevant settings file:

- **User (global):** `~/.claude/settings.json`
- **Project (shared):** `.claude/settings.json`
- **Local (git-ignored):** `.claude/settings.local.json`

Delete the skill from the `enabledPlugins` section.

### Verify

```bash
claude plugin list
```

## Repository Structure

```
.
├── .claude-plugin/          Plugin manifest
├── assets/                  Media files
├── skill/
│   ├── SKILL.md             Skill entry point
│   └── references/          Framework and examples
├── AGENTS.md                Agent navigation guide
├── CHANGELOG.md             Version history
├── CONTRIBUTING.md          Contribution guidelines
└── LICENSE                  MIT
```

## License

MIT
