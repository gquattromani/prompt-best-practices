# Notice

This file collects third-party attributions, trademark acknowledgments, and references that inform the content of this repository. It complements `LICENSE`, which covers only the text authored for this project: the MIT grant does not extend to the third-party material quoted below, which stays under its own terms. Anyone redistributing or citing the project can see the provenance at a glance here.

## Third-party content and references

### Anthropic — prompting best practices

The 7-component framework in `skills/prompt-best-practices/references/framework.md` and `skills/prompt-best-practices/references/component-definitions.md` is derived from [Anthropic's prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices). Each component maps to a specific section of that documentation, and short verbatim quotations (1-2 sentences) appear alongside each mapping for educational and critical purposes, always with attribution and a direct link to the source section.

The `claude-considerations.md`, `claude-opus-5.md`, `claude-fable-5.md`, `grounding-techniques.md`, and `maintenance.md` files under `skills/prompt-best-practices/references/` similarly include short attributed quotations from the same source, including the per-model [Prompting Claude Opus 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5) and [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) pages. In both per-model files the quotations proper are the short passages in double quotes; the instruction blocks and backticked snippets are condensed adaptations of the sample prompts published on those pages, written in this project's own words rather than reproduced. Each file states that basis in its own header, attributes the page it draws on, and says that it does not substitute for it.

The Anthropic documentation pages carry no licence, copyright, or terms notice, so there is no open licence to rely on and no permission was granted: these quotations are used in the belief that they qualify as fair use in the United States and as permitted quotation under the instruments that govern it elsewhere: Article 10 of the Berne Convention, Article 5(3)(d) of Directive 2001/29/EC in the European Union, and Article 70 of the Italian Copyright Act. Fair use is a defence assessed case by case, not a permission granted in advance, so what this project can state is the practice it follows — the one those provisions ask of a quotation: keep it short, attribute it, link the section it comes from, and use it to support the project's own analysis rather than in place of it. No quotation substitutes for the original page, which remains freely accessible at the link above.

### OpenAI — GPT-5.6 model guidance

The `skills/prompt-best-practices/references/codex-considerations.md` file references OpenAI's official [GPT-5.6 model guidance](https://developers.openai.com/api/docs/guides/prompt-guidance-gpt-5p6) and the [Codex / GPT models overview](https://learn.chatgpt.com/docs/models). Short attributed quotations appear in the file for educational purposes, under the same fair-use / right-of-quotation basis described above.

### Google — Gemini prompting guidance

The `skills/prompt-best-practices/references/gemini-considerations.md` file draws on Google's official [prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies) and [Gemini 3 developer guide](https://ai.google.dev/gemini-api/docs/gemini-3). Unlike the sources above, these pages carry an open licence: each footer reads "Except as otherwise noted, the content of this page is licensed under the Creative Commons Attribution 4.0 License", with code samples under Apache 2.0 — [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), confirmed at the source on 2026-08-19. The quotations in that file are used on that basis, attributed to Google, each linked to the page it comes from. They are reproduced verbatim; where one is abridged the omission is marked with an ellipsis, and no altered wording is presented as Google's. No Google code sample is reproduced, so the separate licence those carry does not apply here — the Markdown prompt skeleton in that file is this project's own, as is all surrounding analysis.

### xAI — Grok documentation

The `skills/prompt-best-practices/references/grok-considerations.md` file references xAI's official [Grok 4.6](https://docs.x.ai/developers/grok-4-6), [models](https://docs.x.ai/developers/models), [reasoning](https://docs.x.ai/developers/model-capabilities/text/reasoning), and [Grok Build](https://docs.x.ai/build/overview) documentation, together with xAI's published prompt-engineering guide for its coding model. Short attributed quotations appear in the file for the same purpose and on the same basis.

### GitHub — Copilot supported-models list

The host-to-vendor map in `skills/prompt-best-practices/references/runtime-detection.md` cites GitHub's [supported AI models in GitHub Copilot](https://docs.github.com/en/copilot/reference/ai-models/supported-models) page as evidence that Copilot exposes a multi-vendor model picker. No model list is reproduced.

### Contributor Covenant

`CODE_OF_CONDUCT.md` is an adaptation of the [Contributor Covenant, version 2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/), which is distributed under the [Creative Commons Attribution 4.0 International license (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). It reproduces none of the Covenant's text: it states the adopted principles in this project's own words, links to the full Covenant for the complete version, and replaces the Covenant's enforcement ladder with a simplified enforcement section. Attribution and that modification are both stated in the file itself, as CC BY 4.0 requires of an adaptation.

## Trademarks

The product, service, and model names used in this repository are trademarks or registered trademarks of their respective owners. They appear here exclusively to describe runtime compatibility, cite published documentation, or identify tools that integrate with the Agent Skills standard. No ownership is claimed over any of them, and no sponsorship, endorsement, or affiliation is claimed or implied.

The names used are: Claude, Claude Code, Claude Opus, Claude Sonnet, Claude Haiku, Claude Fable, Claude Mythos, OpenAI, ChatGPT, GPT-5.6, Sol, Terra, Luna, Codex, GitHub, GitHub Copilot, Cursor, Windsurf, Gemini, Gemini CLI, Antigravity, Jules, Grok, Devin, Kiro, Zed, Amp, Sourcegraph, JetBrains, Junie, Qoder, Visual Studio, Visual Studio Code, Moonshot AI, Cline, OpenCode, OpenClaw, Swival, CodeWhale, pi, Aider, Hermes Agent, and skills.sh.

Each name belongs to the organization that owns it. This project is independent and is not affiliated with, sponsored by, or endorsed by any of them, and all trademark rights remain with their respective owners. This notice deliberately does not assign each mark to a named legal entity: corporate ownership changes, and a stale attribution would be worse than none.

## Reporting attribution or trademark issues

If you believe attribution is missing, inaccurate, or that a trademark is used incorrectly, please open an issue describing the problem, or contact the maintainer privately following the process in `SECURITY.md`. Corrections will be made promptly.
