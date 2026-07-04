---
name: delegate
description: Turn Claude Code into a multi-model orchestration and delegation system. FIRST discovers which model CLIs and Claude models are available on THIS machine, then builds a customized "right model for the right job" routing config plus opt-in subagents that delegate work to external models (Codex/GPT, Gemini, OpenRouter, Ollama, local, etc.) and report results back. Use when a user wants to set up model delegation, route tasks to cheaper or other-provider models, offload token-hungry work, or orchestrate subagents across providers.
---

# 🔀 /delegate — Multi-Model Orchestration & Delegation

Turn Claude Code into an **orchestrator**: it hands the right work to the right
model — cheap models for bulk, premium for taste, local for private, other
providers for breadth — then reports results back. Two ideas underpin it:

1. **Right model for the right job** — route by cost / intelligence / taste, and
   *escalate rather than cheap out* when output misses the bar.
2. **Delegate via CLIs** — Claude Code shells out to model CLIs and reports back;
   an opt-in subagent wraps each lane.

> This skill is **self-customizing**. It does NOT hardcode one person's model
> roster. It discovers what *you* actually have and builds your config around it.

---

## 🛑 FIRST RUN — SELF-CUSTOMIZATION (do this before anything else)

If `~/.claude/CLAUDE.md` has no "Model orchestration" section **and**
`~/.claude/agents/` has no delegate wrappers, this skill is unconfigured for this
machine. Execute the four steps in order. Do not skip discovery and paste a
generic roster — the whole point is to fit the user's actual setup.

### Step 1 — DISCOVER what is on THIS machine
Follow [`references/discovery.md`](references/discovery.md). Detect, without
assuming:
- External model CLIs on PATH: `codex`, `gemini`, `agy`, `ollama`, `llm`,
  `aider`, and any others.
- Local models: `ollama list` (if present).
- Which Claude models *this* Claude Code can use (current aliases + any versioned
  models exposed in the picker/config).
- Provider API keys present — **env var NAMES ONLY. Never read, print, echo, or
  store a key's value.**

Report a short table of what's available. That table is the raw material for the
routing config.

### Step 2 — ASK the user to calibrate (don't invent their economics)
Model rankings are personal — they depend on the user's plans and limits, not
list price. Ask:
1. **Which subscriptions/plans do you have?** (e.g. a Gemini subscription, a
   ChatGPT/Codex plan, OpenRouter credits, local GPU.) This decides what is
   "effectively free" vs metered for *them*.
2. **Priority when models conflict** — cost, or output quality?
3. **Any models to avoid** (unproven/untrusted for them), or a hard "never"?
4. **Scope**: global config (`~/.claude/`, every project) or one project
   (`.claude/`)? Global is convenient; project-scoped keeps unrelated/brownfield
   repos unaffected.

Fill the routing table's cost/priority from these answers. Do NOT copy example
numbers from the template.

### Step 3 — BUILD the customized setup
- For each CLI found in Step 1, copy the matching template from
  [`templates/agents/`](templates/agents/) into the chosen agents dir. Skip lanes
  for tools the user does not have.
- Write the routing framework from [`templates/CLAUDE.md`](templates/CLAUDE.md)
  into the chosen CLAUDE.md, filled with THIS user's models + priorities.
- Pin wrapper subagents to the user's **cheapest *proven* Claude model** (ask
  which — the wrappers only craft prompts and summarize, so they should be cheap).
- (Windows) if `codex` or another tool isn't on PATH, see the shim note in
  [`references/provider-setup.md`](references/provider-setup.md).

### Step 4 — VERIFY
Smoke-test one round-trip per configured lane (see
[`references/cli-invocations.md`](references/cli-invocations.md) → "Smoke test").
Each lane should return a correct, model-identified answer before you declare the
setup done. Report the results.

---

## Core principles (the routing ethos)
- **Defaults, not limits.** Judge the OUTPUT, not the price tag. If a cheaper
  model underdelivers, rerun on a stronger one without asking — escalating costs
  less than shipping mediocre work.
- **When axes conflict for anything that ships: intelligence > taste > cost.**
- **Bulk / mechanical / clear-spec** (implementation to spec, data wrangling,
  migrations, investigation) → cheapest capable model (often a GPT/Codex or a
  local model).
- **User-facing** (UI, copy, API design) or top-quality output → highest-taste model.
- **Reviews** → a couple of strong models, optionally one from a *different
  provider* for an independent lens.
- **Research / large-context / web** → a big-context model (e.g. Gemini).
- **Never** route to a model the user flagged as untrusted or "never".

## Delegation mechanics (hard-won — full detail in references/cli-invocations.md)
- **Close stdin or many CLIs hang.** Append `</dev/null` (bash / macOS / Linux /
  Git-Bash) or `<nul` (Windows `cmd`). This is the #1 cause of "it just hangs".
- **Prefer file output over stdout** when a CLI drops output on a non-TTY pipe
  (some agentic CLIs do): have it write to a file and read the file.
- **Keep delegation OPT-IN.** Subagents should fire only on explicit request and
  never auto-delegate — this protects governed/brownfield repos from silently
  routing work to an external model.
- **Dynamic spawns take aliases only.** Claude Code's on-the-fly subagent spawn
  accepts current model aliases; to run a subagent on a *versioned* model, PIN it
  in an agent file.
- **Sandbox awareness.** Claude Code's tool shell may be sandboxed/isolated from
  the host: tools *installed* by the agent may not reach the user's real machine,
  and interactive logins done in a terminal may not be visible to the agent's
  shell. Have the user install host tools and authenticate themselves; verify.
- **Report back, don't dump.** A wrapper returns a tight synthesis + the model
  used, not the raw transcript.

## Invoking (after setup)
- Natural language: "delegate this to codex", "have gemini research X",
  "use a local model for this".
- Explicit: `@codex-delegate`, `@gemini-research`, `@model-delegate`.

## Extending — add a new provider lane
Copy [`templates/agents/model-delegate.md`](templates/agents/model-delegate.md),
swap in the new CLI + flags, keep it opt-in and reliable-stdout, then add a row to
your CLAUDE.md routing table. The `llm` CLI (one tool, many providers) is the
lowest-effort way to add breadth.

## Credits
Inspired by [@theo](https://x.com/theo) (t3.gg)'s posts on model-tiering for
agent orchestration — keeping a `CLAUDE.md` section that prioritizes different
models for different work, and teaching Claude Code to use Codex (and other CLIs)
as delegation fallbacks for token-hungry tasks (implementation, computer-use,
codebase analysis) while the primary model orchestrates.

This skill generalizes that idea into a **self-discovering** setup (it detects
each user's own models/CLIs instead of hardcoding a roster) and hardens the CLI
delegation with the non-TTY / stdin-hang / versioned-model-pinning / sandbox-
isolation / opt-in lessons learned making it reliable in practice.
