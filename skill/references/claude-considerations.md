# Claude-Specific Considerations

These guidelines apply specifically when using Claude models. They are derived from Anthropic's official documentation and address behaviors particular to the Claude model family.

## Claude 4.6

Derived from [Overthinking and excessive thoroughness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#overthinking-and-excessive-thoroughness), [Overeagerness](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#overeagerness), [Tool usage](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#tool-usage), and [Minimizing hallucinations in agentic coding](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices#minimizing-hallucinations-in-agentic-coding).

1. **Reduce anti-laziness prompting.** "Tools that undertriggered in previous models are likely to trigger appropriately now. Instructions like 'If in doubt, use [tool]' will cause overtriggering." Use natural language instead.

2. **Prefer general instructions over prescriptive steps.** "A prompt like 'think thoroughly' often produces better reasoning than a hand-written step-by-step plan." Claude's adaptive thinking frequently exceeds what a human would prescribe.

3. **Watch for over-engineering.** "Claude Opus 4.6 [has] a tendency to overengineer by creating extra files, adding unnecessary abstractions, or building in flexibility that wasn't requested." Keep Output specifications focused and explicit about scope.

4. **XML tags are your strongest formatting tool.** Claude parses XML tags unambiguously. Use them consistently for any prompt with more than 2 components.

5. **Don't force deliberation phases on clear tasks.** Claude 4.6 handles well-defined tasks better without forced clarification or planning steps. For ambiguous or high-stakes tasks, you can optionally ask Claude to outline its approach before executing — but this is a technique, not a required component.

## Update procedure

When Anthropic releases a new model generation (e.g., Claude 5.x), update this file with the new guidance and rename the relevant section accordingly.
