# Model orchestration — right model for the right job
# TEMPLATE: fill the <PLACEHOLDERS> from discovery + the user's answers.
# Place at ~/.claude/CLAUDE.md (global) or <project>/.claude/CLAUDE.md (scoped).

Delegation is OPT-IN: only on explicit request ("use codex", "have gemini
research", "@codex-delegate"). NEVER auto-delegate during normal work. Don't grep
the repo or improvise flags — they're below.

## Roster (this machine's available + approved models)
Calibrate `cost` to what the USER actually pays/limits (not list price). State
whether higher means cheaper or pricier and be consistent.

| Model | cost | intel | taste | Reached via | Right job |
|-------|------|-------|-------|-------------|-----------|
| <cheap-bulk model, e.g. GPT via codex> | <n> | <n> | <n> | `codex exec` | bulk, mechanical, clear-spec impl, data, investigation |
| <cheap proven Claude, e.g. a specific version> | <n> | <n> | <n> | Agent `model:` (pin full id) | light Claude-flavored work, wrapper orchestration |
| <strong Claude> | <n> | <n> | <n> | main loop / Agent `model:` | hard reasoning, orchestration |
| <premium/taste model> | <n> | <n> | <n> | Agent `model:` | UI/copy/API design, final polish, deep reviews |
| <big-context model, e.g. Gemini> | — | — | — | `gemini -p` / `llm` | research, large-context, web |
| <local model, if any> | free | <n> | <n> | `ollama run` | private/offline bulk |

Do NOT use: <models the user flagged untrusted/never>.
CONSTRAINT: dynamic Agent-tool spawns accept only current aliases; to run a
subagent on a versioned model, PIN the full id in an agent file.

## How to apply (defaults, not limits)
- Judge the OUTPUT, not the price. If a cheaper model underdelivers, rerun on a
  stronger one without asking — escalating beats shipping mediocre work.
- When axes conflict for anything that ships: intelligence > taste > cost.
- Bulk / mechanical / clear-spec → <cheapest capable>.
- Light Claude-flavored work → <cheap proven Claude>.
- User-facing (UI, copy, API design) or top-quality → <premium/taste model>.
- Hardest / most ambiguous reasoning → <highest-intelligence model>.
- Reviews of plans/impl → two strong models; optionally one other-provider lens.
- Research / read-everything / web → <big-context model>.

## Verified invocations (external CLIs; close stdin or they hang)
- <cheap-bulk>:  codex exec -s read-only "<prompt>" </dev/null
                 (workspace-write ONLY when the user authorizes edits · codex exec review)
- <big-context>: gemini -p "<self-contained query>" </dev/null
- other models:  llm -m <model> "<prompt>" </dev/null
- local:         ollama run <model> "<prompt>"
- Parallel research → one process per topic, redirect each to its own file:
                 gemini -p "<bounded query>" </dev/null > <dir>/<topic>.md
  Bound every prompt to the project's actual architecture + invariants.
- Windows cmd uses `<nul` instead of `</dev/null`.

## Mechanics
- Non-Claude models are reached ONLY via their CLIs. In subagents the model param
  takes Claude models only, so a non-Claude model runs through a thin wrapper that
  shells out to the CLI and returns the result (see the delegate wrappers).
- Claude models run via the Agent/subagent `model` parameter (alias, or full id
  when pinned in an agent file).

Wrapper subagents (`@codex-delegate`, `@gemini-research`, `@model-delegate`) are
pinned to <cheapest proven Claude> — they only craft prompts + summarize.
