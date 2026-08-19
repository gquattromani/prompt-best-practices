# Security Policy

## Reporting a Vulnerability

If you discover a security issue in this skill — for example a prompt-injection pattern that bypasses the intended activation rules, a suggestion that leaks private data, or any other behavior that could harm users — please report it privately.

**Do not open a public issue.** Instead:

1. Use GitHub's [private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability) on this repository, or
2. Contact the maintainer directly via the email listed in the Git commit history.

You can expect an initial response within 7 days. Valid reports will be credited in the changelog unless you prefer to remain anonymous.

## Scope

This skill is a collection of Markdown instructions and manifests for AI agents. Nothing in it makes a network call and it has no dependencies; the only code that runs is a single `echo` per host in the optional activation hooks (`hooks/`), which prints a reminder and stores nothing. The security surface is:

- **Prompt-injection patterns** in the dialogue examples that an attacker could reuse to bypass an agent's own guardrails
- **Misleading guidance** that would lead an agent to recommend insecure code
- **Broken activation rules** that cause the skill to run on prompts where it shouldn't

Reports outside this scope (e.g., vulnerabilities in Claude Code, Codex, or another host agent) should be reported to the respective vendor.

## Security Review

A manual audit of the repository was performed to verify that the skill and its documentation do not promote risky or dubious developer activity. The following patterns were searched across all tracked files and **not found**:

| Category | Result |
|---|---|
| Destructive shell commands (`rm -rf`, `sudo`, `chmod`) | Not present |
| Git-history rewriting or hook bypass (`--no-verify`, `--force`, `git reset --hard`, `git push --force`) | Not present |
| Pipe-execution patterns (`curl ... \| sh`, `wget ... \| bash`) | Not present |
| Hardcoded secrets, tokens, or passwords | Not present — the only token/password references are pedagogical examples teaching bcrypt hashing, token expiry, and rate limiting |
| Instructions to disable tests, linting, or type-checking | Not present |
| Executable `eval` / `exec` / `os.system` / `subprocess` calls in repo content | Not present |
| Instructions telling the agent to perform destructive operations without user confirmation | Not present |

This review should be re-run when the repository receives substantive changes (new skill files, new examples, new installation instructions).
