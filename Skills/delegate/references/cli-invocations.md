# CLI invocations — verified patterns & gotchas

Cross-provider invocation recipes for delegation. The recurring failure modes are
**stdin hangs** and **non-TTY stdout drops** — the notes below are how to avoid
them. Test any new CLI's exact flags with `--help` before relying on it.

## Universal rules
- **Close stdin** or the CLI may block waiting for a console:
  - bash / macOS / Linux / Git-Bash:  `<CLI> ... </dev/null`
  - Windows `cmd`:  `<CLI> ... <nul`
- **Self-contained prompts.** A delegated CLI does NOT see your conversation.
  State the goal, the exact files/paths, constraints, and what "done" means.
- **Report back, don't dump.** Capture output, sanity-check it, return a tight
  summary + which model ran — not the raw transcript.

## Codex (OpenAI GPT family) — `codex`
Non-interactive:
```bash
codex exec -s read-only "<self-contained prompt>" </dev/null      # analysis / investigation
codex exec -s workspace-write "<prompt>" </dev/null               # only when edits are authorized
codex exec review                                                 # review current repo diff
```
- Sandbox policy: prefer `read-only`; escalate to `workspace-write` only on
  explicit authorization; avoid full-access unless the user demands it.
- Requires stdin closed (`</dev/null`) or it waits on "Reading additional input".

## Gemini — `gemini`
Reliable non-interactive stdout, web-capable, large context:
```bash
gemini -p "<self-contained query>" </dev/null
```
Parallel research → one process per topic, each redirected to its own file (avoids
truncation, keeps the model read-only — the shell writes, not the model's tools):
```bash
gemini -p "<bounded query>" </dev/null > research/<topic>.md
```

## Antigravity CLI — `agy`
Spends an Antigravity/Gemini subscription and exposes multiple models, BUT its
**headless stdout is unreliable on a non-TTY pipe** — piped output can vanish.
- It works in a real interactive terminal.
- For automation, either (a) prefer `gemini`/`llm` for the same models, or
  (b) have the *user* run `agy -p "..."` in their terminal and paste results back.
- Do not reach for guard-disabling flags to force headless writes.

## Local models — `ollama`
Free, private, offline:
```bash
ollama run <model> "<prompt>"        # e.g. a local coder or reasoning model
```

## Many providers, one CLI — `llm` (Simon Willison's CLI)
Best way to add breadth (OpenRouter, Anthropic, Gemini, Mistral, local, ...):
```bash
llm -m <model> "<prompt>" </dev/null
# e.g. llm -m openrouter/<vendor>/<model> "..."   (Grok, DeepSeek, Qwen, etc.)
```
Keys live in `llm`'s keystore (`llm keys set <provider>`), never in agent files.

## Smoke test (run one per configured lane)
Dispatch the same verifiable task to each lane and confirm a correct,
model-identified answer:
```bash
# example task with a checkable answer (12th Fibonacci, F1=F2=1, is 144)
codex exec -s read-only "Reply on ONE line: MODEL=<your model>; FIB12=<12th Fibonacci, F1=1 F2=1>" </dev/null
gemini -p "Reply on ONE line: MODEL=<your model>; FIB12=<12th Fibonacci, F1=1 F2=1>" </dev/null
```
A good result names a *different* model per lane and returns the right number —
proving the orchestrator is really reaching distinct backends.

## Sandbox caveat
If Claude Code's tool shell is sandboxed, it may not share the host's PATH,
installs, or interactive logins. Symptoms: a freshly "installed" tool isn't found
in the user's own terminal, or an authenticated CLI hangs on auth for the agent.
Fix: the user installs host tools and logs in themselves; the agent verifies via a
smoke test rather than assuming.
