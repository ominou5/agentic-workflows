# Discovery — detect this machine's models & CLIs

Goal: build the routing config from what the user *actually has*, not assumptions.
Run these read-only checks and summarize the results. **Never read or print the
value of any API key — names only.**

## 1. Which model CLIs are installed?
Check PATH for each known model CLI. Presence = a possible lane.

**bash / macOS / Linux / Git-Bash:**
```bash
for c in codex gemini agy ollama llm aider cursor-agent; do
  p=$(command -v "$c" 2>/dev/null) && echo "FOUND  $c -> $p" || echo "absent $c"
done
```

**Windows PowerShell:**
```powershell
foreach ($c in 'codex','gemini','agy','ollama','llm','aider') {
  $s = (Get-Command $c -ErrorAction SilentlyContinue).Source
  if ($s) { "FOUND  $c -> $s" } else { "absent $c" }
}
```

## 2. Local models (if Ollama present)
```bash
ollama list        # models already pulled and runnable locally, offline & free
```

## 3. Which Claude models can this Claude Code use?
- Ask the user to open the model picker and list what they see (current aliases
  plus any versioned models like older Opus/Sonnet builds).
- Versioned model IDs generally follow a short form (e.g. `claude-<family>-<major>-<minor>`).
  Confirm the exact id string before pinning a subagent to it — a wrong id makes
  the agent fail to load.
- Remember: **dynamic subagent spawns accept only the current aliases**
  (`sonnet`/`opus`/`haiku`/`fable`-style). To use a *versioned* model in a
  subagent you must pin the full id in an agent file.

## 4. Provider credentials present (NAMES ONLY)
Detect which providers are configured, so routing knows what's reachable. Print
only the variable NAMES that exist — never the values.

**bash:**
```bash
env | grep -oiE '^(OPENAI|ANTHROPIC|GEMINI|GOOGLE|OPENROUTER|XAI|GROK|DEEPSEEK|MISTRAL|GROQ|TOGETHER|FIREWORKS|COHERE)[A-Z_]*=' | sed 's/=$//' | sort -u
```

**Windows PowerShell:**
```powershell
Get-ChildItem Env: | Where-Object { $_.Name -match '^(OPENAI|ANTHROPIC|GEMINI|GOOGLE|OPENROUTER|XAI|GROK|DEEPSEEK|MISTRAL|GROQ|TOGETHER|FIREWORKS|COHERE)' } | Select-Object -ExpandProperty Name
```
Also check the `llm` keystore if `llm` is installed: `llm keys` (lists provider
names that have keys set — not the keys themselves).

## 5. Produce a capability table
Summarize for the user, e.g.:

| Lane | Tool found | Auth | Notes |
|------|-----------|------|-------|
| GPT / Codex | `codex` ✅ | (existing login) | agentic exec, computer-use |
| Gemini | `gemini` ✅ | oauth | web-capable research, big context |
| Local | `ollama` ✅ (3 models) | n/a | free, private, offline |
| Multi-provider | `llm` ✅ | keys: openrouter | breadth via one CLI |
| Claude | native | plan | aliases + versioned builds in picker |

Then proceed to Step 2 (calibrate) in `SKILL.md`.
