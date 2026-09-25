# Activation Regression Fixtures

## Purpose

Reference prompts with expected activation outcomes. A contributor modifying `SKILL.md`, `framework.md`, `component-definitions.md`, or `component-rubrics.md` must verify that every fixture still produces its expected outcome; a contributor modifying `runtime-detection.md`, `universal-baseline.md`, or any vendor file must verify fixture set H. This is the regression test for the skill's behavior.

These fixtures address the gap flagged in `SECURITY.md` > Scope > "Broken activation rules". Run them manually against the skill (or via an automated harness if one is added later); they are the contract the skill has to honor.

## How to run

For each fixture:

1. Invoke the skill as described in the `Trigger` column (slash command or auto-mode prompt).
2. Observe whether the skill activates, fast-tracks, or declines.
3. Compare against `Expected outcome`.
4. If any fixture fails, the change is a regression — fix or document before merging.

## Legend

- `ACTIVATE_FULL` — skill runs Step 1 diagnosis and full dialogue.
- `ACTIVATE_LIGHT` — skill runs Step 1, but the tier cap limits the dialogue to ≤2 questions.
- `FAST_TRACK` — skill runs Step 1 and proceeds to execute without dialogue (e.g., because the user said "skip").
- `DO_NOT_ACTIVATE` — skill does not run; the host agent answers directly.

---

## Fixture set A — Auto-mode prompts (no slash command)

| # | Prompt | Tier | Expected outcome | Rationale |
|---|---|---|---|---|
| A1 | `Rename the variable userList to users in UserViewModel.swift` | Quick (micro-task) | `DO_NOT_ACTIVATE` | Precise code-level micro-task. Codebase supplies context. |
| A2 | `Fix the typo in line 42 of README.md` | Quick (micro-task) | `DO_NOT_ACTIVATE` | Typo fix. No ambiguity. |
| A3 | `What is the Observer pattern in Swift?` | N/A (question) | `DO_NOT_ACTIVATE` | Conversational question, not an execution request. |
| A4 | `Write me an email` | Quick → Standard (depends on stakes) | `ACTIVATE_FULL` | Execution request; no success criterion; no output shape. |
| A5 | `Write a commit message for the staged diff` | Quick | `ACTIVATE_LIGHT` | Execution request; tier cap limits dialogue to ≤2 questions. |
| A6 | `Build me an endpoint to reset user passwords` | Complex | `ACTIVATE_FULL` | Security-sensitive, multi-subsystem, needs full dialogue. |
| A7 | `Migrate the auth middleware from sessions to JWT` | Complex | `ACTIVATE_FULL` | Cross-cutting refactor; stakes high. |
| A8 | `Just write me a haiku about autumn, don't overthink` | Quick | `DO_NOT_ACTIVATE` | User explicitly opted out of optimization. |
| A9 | `Add the missing import for React in App.tsx` | Quick (micro-task) | `DO_NOT_ACTIVATE` | Trivial, no ambiguity. |
| A10 | `<role>You are a senior Swift engineer</role><task>Write tests for PaymentService with 100% branch coverage</task><output_spec>Swift Testing, clear test names</output_spec><constraints>Follow tests/OrderServiceTest.swift pattern</constraints>` | Complex (already structured) | `DO_NOT_ACTIVATE` | 5+ components present at `[OK]`; meets complex-tier threshold. |
| A11 | `Write me an email` submitted on a host where the activation reminder is delivered (a `UserPromptSubmit` hook on Claude Code, Codex or Copilot CLI, or an always-on rule file elsewhere) | Standard | `ACTIVATE_FULL` **without the user typing the slash command** | This is the regression test for the failure the reminder exists to fix: the `AUTO-ACTIVATE` rules live inside `SKILL.md` and are invisible until the skill is chosen, so before 0.9.0 the same prompt usually just got executed. If A11 fails, check the hook fired (`/hooks`) before touching the activation rules. |
| A12 | `What is the Observer pattern in Swift?` on the same host, reminder delivered | N/A (question) | `DO_NOT_ACTIVATE`, and the reminder is not mentioned to the user | The reminder carries the skip conditions, so a question must still not activate. A host where A11 passes but A12 fails means the reminder is being read as a command rather than as a condition. |

## Fixture set B — Slash-command invocation

The `MANDATORY on slash-command invocation` rule applies. Step 0 always runs; Fast-Track Exit is offered in Step 1.

| # | Prompt | Expected outcome | Rationale |
|---|---|---|---|
| B1 | `/prompt-best-practices write me an email` | `ACTIVATE_FULL` | Standard tier, low structure. |
| B2 | `/prompt-best-practices rename foo to bar` | `ACTIVATE_LIGHT` | Slash = assess; tier = quick, cap 2 questions. Fast-Track Exit offered. |
| B3 | `/prompt-best-practices what is recursion?` | `ACTIVATE_LIGHT` | Slash = assess; skill should note the prompt is a question and offer to skip. |
| B4 | `/prompt-best-practices` (user then replies `skip` to Step 1) | `FAST_TRACK` | Fast-Track Exit honored. |
| B5 | `/prompt-best-practices <full 7-component prompt>` | `ACTIVATE_LIGHT` | Slash = assess; diagnostic shows all `[OK]`; skill should immediately offer to execute. |

## Fixture set C — Ambiguity and edge cases

| # | Prompt | Expected outcome | Rationale |
|---|---|---|---|
| C1 | `Write me an email. Just execute, no questions.` | `DO_NOT_ACTIVATE` | Explicit opt-out. |
| C2 | `Write me an email about the Q2 numbers. It should be under 200 words, formal tone, and get them to confirm the meeting on Thursday.` | Standard, meets threshold | `DO_NOT_ACTIVATE` | Task, Output spec, Success criterion all present → ≥3 components. |
| C3 | `Analyze these sales data and give me useful insights` | Complex (open-ended) | `ACTIVATE_FULL` | "Useful" is not a success criterion; many unknowns. |
| C4 | `Debug why the integration tests are flaky` | Complex | `ACTIVATE_FULL` | Open-ended investigation; benefits from scoping. |
| C5 | `Add a null-check at line 88 of UserService.ts` | Quick (micro-task) | `DO_NOT_ACTIVATE` | Precise, small, context from codebase. |
| C6 | `Refactor UserService to be cleaner` | Standard / Complex | `ACTIVATE_FULL` | "Cleaner" is subjective; needs success criterion. |
| C7 | User starts dialogue, gives one answer, then replies `let's go` to next question. | Dialogue stops early | Skill must honor impatience signal and build with what it has. |

## Fixture set D — Runtime-specific behavior

These fixtures verify that Step 3 (Build Final Prompt) picks the right Structure format per runtime.

| # | Context | Expected format in final prompt | Rationale |
|---|---|---|---|
| D1 | Target runtime: Claude (Opus 5.5 default target, Claude Code) | XML tags | `component-definitions.md` > Component 7 default; `claude-considerations.md` > "Behaviors shared across the current family" § 1. |
| D2 | Target runtime: Codex (`gpt-6-sol`) | Markdown headings or XML, both acceptable | `codex-considerations.md` > "Other Codex-specific behaviors" § 4. |
| D3 | Target runtime not specified, or a multi-model host with no visible selection | XML tags (safe default), plus the universal baseline for everything else | `component-definitions.md` > Component 7 > Runtime-specific defaults, and `universal-baseline.md` > rule 2: the downside is asymmetric — a hidden Claude loses its strongest formatting tool under Markdown, while XML costs nothing on the other three vendors. Fixture H5 covers the rest of the baseline. |
| D4 | Target runtime: Claude Opus 5.5, Claude Fable 5.1, or GPT-6, prompt over-specified (padded examples, repeated rules) | Skill trims to the smallest sufficient prompt | `framework.md` > "Calibrating for frontier models"; leaner prompts win on all current frontier models. |

## Fixture set E — Model-gated content

These fixtures verify that Step 3 applies per-model guidance instead of a single Claude-wide default. They are the regression test for `claude-considerations.md` > "Pick the model before you tune": the failure mode is flattening the divergence, so E1 and E2 must not both resolve the same way.

| # | Context | Expected behavior | Rationale |
|---|---|---|---|
| E1 | Target runtime: Claude Opus 5 or Opus 5.5. User's original prompt contains "and double-check your answer before finalizing". | The verification clause is **removed** from the final prompt, with a one-line note that it was dropped. | `claude-opus-5-5.md` > Inherited § 2; `grounding-techniques.md` technique 3 is model-gated. Rewording it is also a failure — the documented fix is removal. |
| E2 | Target runtime: Claude Fable 5.1, long autonomous run. | The verifier instruction is **kept**, preferring a fresh-context verifier subagent over self-critique. | `claude-fable-5.md` § 8. Same clause, opposite call — the divergence must survive. |
| E3 | Target runtime: Claude Opus 5.5, quick-tier prompt with no length or tone stated. | `<output_spec>` includes an explicit length/conciseness line even at quick tier. | `claude-opus-5-5.md` > Inherited § 1: default responses run long and `effort` does not shorten visible output, so Component 5 is load-bearing here. |
| E4 | Target runtime: Claude Opus 5.5, harness supports subagents. | Constraints include a delegation cap, not a delegation nudge. | `claude-opus-5-5.md` > Inherited § 4. A "use subagents freely" line carried over from Opus 4.8, Fable, or GPT-6 guidance is a regression. |
| E5 | Target runtime: Claude Opus 5.5, chat system prompt that contains "Think carefully before answering." | The line is **removed**, with a one-line note; depth is left to `effort`. | `claude-opus-5-5.md` > "What changes on Opus 5.5" § 3. Rewording it ("reason step by step") is also a failure, and asking for the reasoning in the response adds a `reasoning_extraction` refusal risk. |
| E6 | Target runtime: Claude Fable 5.1. The original prompt carries a block that forbids bullet points, headers, and bold. | The block is removed or replaced by a rule that says *when* a list helps, with a one-line note. | `claude-fable-5.md` > "What changes on Fable 5.1" § 5: the model already formats less, so the block suppresses structure the content needs. On Opus 5.5 the same block is still the general guide's technique for minimizing markdown — the divergence must survive. |

## Fixture set F — Loadability (mechanical checks)

These fixtures protect the skill from a failure mode that has nothing to do with its logic: content that is present but never actually read. A reference file over budget can be truncated or skipped by the loading agent, and a cross-reference that points at a moved section silently degrades into no guidance at all. F1-F3 are deterministic shell checks — run them from the repository root before merging any change that adds, splits, renames, or grows a file.

| # | Check | Command | Expected |
|---|---|---|---|
| F1 | Token budgets are respected | `wc -w skills/prompt-best-practices/SKILL.md skills/prompt-best-practices/references/*.md` | `SKILL.md` ≤ 3500 words (~5000 tokens); every reference file ≤ 2700 words (~4000 tokens). |
| F2 | Every reference file declares its purpose | `rg -c '^## When to Use' skills/prompt-best-practices/references/*.md` | Every file reports `1`. A file with no count is missing the header an agent uses to decide whether loading it is worth the tokens. |
| F3 | Every path named in `SKILL.md` resolves | `rg -o 'references/[a-z0-9-]+\.md' skills/prompt-best-practices/SKILL.md \| sort -u \| while read -r p; do [ -f "skills/prompt-best-practices/$p" ] \|\| echo "MISSING: $p"; done` | No output. Reference paths in `SKILL.md` must always carry the `references/` prefix, or an agent resolving them against the working directory will fail to open the file. |
| F4 | A missing reference does not abort the workflow | Rename one reference file temporarily, then invoke the skill on a standard-tier prompt. | The skill completes Step 0 through Step 3 using `SKILL.md` alone and states in one line that a reference was unavailable. It must not decline, and it must not silently skip the diagnosis. |

## Fixture set H — Runtime routing (behavioral)

These fixtures protect the vendor-routing layer added in 0.8.0. The failure they exist to catch is silent: a prompt that is *well structured for the wrong vendor* looks correct in review and underperforms in use. `references/runtime-detection.md` is the contract; each row below is one of its rules made testable. Run them when `runtime-detection.md`, `universal-baseline.md`, or any vendor file changes.

| # | Situation | Expected behavior | Rationale |
|---|---|---|---|
| H1 | The skill runs in Claude Code. The user names no model. | Final prompt uses XML tags, puts long data at the top with the task at the end, states response length in `<output_spec>`, and carries no self-check clause. | Claude Code is single-vendor, so the host resolves the family; Opus 5.5 is the default target and its verification clause must be absent, not softened (`claude-opus-5-5.md` > Inherited § 2). |
| H2 | The skill runs in Gemini CLI or Antigravity with the default picker. | Markdown headings or XML used consistently; data first with the instruction last and an anchor phrase; an explicit length and tone line; **no** temperature or sampling instruction. | Gemini 3.x is terse by default and the vendor says to remove sampling parameters from requests — a carried-over `temperature: 0.2` is a regression, not a preference (`gemini-considerations.md`). |
| H3 | Same as H2, but the task is heavy reasoning (a proof, a tricky migration plan). | A "think very hard before answering" style nudge is *allowed* in the prompt text. | Google explicitly endorses it; the same line would be waste on Claude and GPT-6. This is the one row where the cross-vendor divergence flips in favor of more text. |
| H4 | The skill runs in Grok Build. | Role is kept, Constraints spell out edge cases, context names specific paths, and nothing asks for tool calls in an XML envelope. | xAI asks for a thorough system prompt; the lean-prompt reflex from the other three vendors under-specifies here (`grok-considerations.md` > headline). |
| H5 | The skill runs in GitHub Copilot, Cursor, or another multi-model host, and the selected model is not visible. | The universal baseline is used — XML tags, context first, at most one example, explicit length, no vendor-specific parameter text, no self-check clause — and the answer states in one line which runtime it assumed. | Guessing a vendor from a multi-model host is the failure this rule prevents. The baseline is a correct output, so the skill must not stall or ask which model is selected. |
| H6 | The skill runs in Claude Code, but the user says "this prompt goes to Gemini 3.1 Pro". | The Gemini branch is used, not the Claude branch. | An explicit statement outranks the host: prompts are frequently written in one agent and executed in another. |
| H7 | Quick-tier prompt in a multi-model host. | No tool call and no dialogue question is spent discovering the model. | Runtime resolution is a routing decision, not a component; the tier caps in `SKILL.md` > Step 2 are for the 7 components only. |
| H8 | The user's original prompt contains "keep temperature low and double-check the result", target resolved to Gemini 3.x. | Both clauses are removed — the sampling instruction because the vendor says to remove sampling parameters, the verification clause because Gemini's documented accuracy mechanism is Search grounding and code execution — with a one-line note that they were dropped. | Two different vendors' *delete, not rewrite* rules landing in the same prompt. Rewording either one is a failure. |
| H9 | A prompt the user says will be reused across Claude, GPT and Gemini. | Built to the universal baseline, with the per-vendor deltas offered as a list rather than baked in. | `universal-baseline.md` > "Upgrading from the baseline" — one portable prompt plus a delta beats three forks. |
| H11 | The skill runs in Codex (GPT-6 default). Code-change task in a repository. | One authorization boundary that *grants* in-scope, reversible work without asking and names what needs confirmation; an explicit writing style in the output spec; test scope stated instead of a verification step; no delegation cap. | `codex-considerations.md` > "What changes on GPT-6": the model asks more and tests more by default. The same prompt copied from H1 (Opus 5.5) would carry a delegation cap and no authorization grant — the cross-vendor divergence must survive. |

**H10 — mechanical: every runtime named in the host map routes to a file that exists.**

```bash
rg -o '`(claude-considerations|codex-considerations|gemini-considerations|grok-considerations|universal-baseline)\.md`' \
   skills/prompt-best-practices/references/runtime-detection.md | tr -d '`' | sort -u \
   | while read -r f; do [ -f "skills/prompt-best-practices/references/$f" ] || echo "MISSING: $f"; done
```

Expected: no output. A host row pointing at a missing file falls back to the baseline silently, which is exactly the "structured for the wrong vendor" failure H1-H4 exist to prevent.

## Fixture set G — Portability (mechanical checks)

These fixtures protect the host adapters described in `docs/agent-portability.md`. An
adapter is a promise that a given platform can load this skill; a manifest that no
longer parses, a rule copy that drifted from `AGENTS.md`, or a path that points at a
moved folder breaks that promise silently — the host just loads nothing. Run all four
from the repository root (bash or zsh; G1 uses process substitution) before merging any
change to `AGENTS.md`, a rule copy, a manifest, or the `skills/` and `commands/` layout.

**G1 — every instruction-tier rule copy matches the canonical body in `AGENTS.md`.**
The canonical body is everything above the `<!-- shared-rule-end -->` marker; copies add
host-specific frontmatter and nothing else.

```bash
body() { sed '/<!-- shared-rule-end -->/,$d' "$1" \
  | awk 'NR==1 && /^---$/ {f=1; next} f && /^---$/ {f=0; next} !f' | awk 'NF'; }
for f in .cursor/rules/prompt-best-practices.mdc \
         .windsurf/rules/prompt-best-practices.md \
         .clinerules/prompt-best-practices.md \
         rules/prompt-best-practices.md \
         .qoder/rules/prompt-best-practices.md \
         .kiro/steering/prompt-best-practices.md \
         .github/copilot-instructions.md; do
  diff -q <(body AGENTS.md) <(body "$f") > /dev/null || echo "DRIFT: $f"
done
```

Expected: no output. On drift, regenerate the copies with the snippet in
`docs/agent-portability.md` > Adapter rule — do not hand-edit one copy, or the next host
to be installed gets different rules than the others.

**G2 — every host manifest parses.**

```bash
for f in plugin.json package.json gemini-extension.json \
         .claude-plugin/plugin.json .claude-plugin/marketplace.json \
         .codex-plugin/plugin.json \
         .qoder-plugin/plugin.json .grok-plugin/marketplace.json \
         .github/plugin/plugin.json .github/plugin/marketplace.json; do
  python3 -m json.tool "$f" > /dev/null || echo "INVALID: $f"
done
python3 -c "import tomllib; tomllib.load(open('commands/prompt-best-practices.toml','rb'))"
```

Expected: no output. A manifest that fails to parse is not a partial install — the host
skips the plugin entirely.

**G3 — one version across every manifest.**

```bash
rg --hidden --no-heading -o '"version": "[^"]+"' -g '*.json' | sort -t: -k2 -u
```

Expected: one distinct version value. The root `plugin.json` carries only a `name` by
Grok's convention, and the two `marketplace.json` files are unversioned by design; every
other manifest must report the same number as `.claude-plugin/plugin.json`.

**G4 — every path a manifest declares exists.**

```bash
rg --hidden --no-heading -o '"(skills|rules|commands|hooks|contextFileName)": "[^"]+"' -g '*.json' \
  | sed 's/.*: "//; s/"$//' | sort -u \
  | while read -r p; do [ -e "$p" ] || echo "MISSING: $p"; done
```

Expected: no output. A manifest pointing at a moved folder loads an empty skill set, which
looks identical to a working install until someone invokes the skill.

**G5 — the activation reminder names the skill and repeats its skip conditions, in every hook file.**
The hook exists to put the activation rule in front of the model before it decides (`docs/agent-portability.md` >
Activation delivery). A reminder that names the skill wrongly, or drops the skip conditions, either fails to
activate or activates on everything.

```bash
for f in hooks/*.json; do
  for needle in prompt-best-practices "execution request" "Skip silently"; do
    grep -q "$needle" "$f" || echo "MISSING in $f: $needle"
  done
done
```

Expected: no output. `prompt-best-practices` must be the skill slug exactly as installed, or the agent cannot
invoke what the reminder names.

**G6 — the hook command runs, prints something, and needs no interpreter.**

```bash
cmd=$(python3 -c "import json; print(json.load(open('hooks/activation-reminder-claude-codex.json'))['hooks']['UserPromptSubmit'][0]['hooks'][0]['command'])")
case "$cmd" in *node*|*python*|*.js*|*.sh*) echo "DEPENDENCY: $cmd";; esac
[ -n "$(sh -c "$cmd")" ] || echo "EMPTY OUTPUT"
```

Expected: no output. A hook that depends on `node` fails silently for Nix and nvm users whose non-interactive
shell has a different PATH, which is the failure mode the plain-`echo` form avoids.

## Known-limitation fixtures

Fixtures where the skill's current heuristics are weak. Document them here so contributors know what not to silently break:

- **L1**: Non-English prompts. The skill should respond in the same language, but tier classification and rubric matching may degrade. If you improve non-English detection, re-verify A1–H11 still pass.
- **L2**: Prompts with inline code blocks that contain XML. The rubric's `[OK]` detection on Component 7 (Structure) must not confuse *content XML* with *prompt-structuring XML*.

---

## Update procedure

When you add a fixture, include the rationale column — it explains *why* the outcome is correct, which prevents other contributors from "fixing" the fixture to match a regression.

When a fixture starts failing, the first move is to diagnose whether the regression is in the skill or the fixture is outdated. Do not update the fixture to match the new (broken) behavior without explicit review.
