# Provider setup — install & auth

Install only the lanes the user wants. **The agent should not run installers that
write to the host if its shell is sandboxed** — have the user run install/login
commands in their own terminal, then verify. Never embed API keys anywhere; each
tool has its own credential store.

## Codex (OpenAI GPT family)
- Install per OpenAI's current instructions for the Codex CLI.
- Auth: sign in with the OpenAI/ChatGPT account per the CLI's flow.
- Verify: `codex --version`, then a `codex exec -s read-only "..." </dev/null` smoke test.

### Windows: stable `codex` shim (optional)
Some installs place `codex.exe` under a versioned hash directory that changes on
update, so it isn't on a stable PATH. A tiny launcher on a PATH directory resolves
the newest install. Create `codex.cmd` on a directory that is on PATH:
```bat
@echo off
setlocal
set "BASE=%LOCALAPPDATA%\<vendor-codex-dir>\bin"
set "EXE="
for /f "delims=" %%d in ('dir /b /ad /o-d "%BASE%" 2^>nul') do (
  if not defined EXE if exist "%BASE%\%%d\codex.exe" set "EXE=%BASE%\%%d\codex.exe"
)
if not defined EXE ( echo codex.exe not found under %BASE% 1>&2 & exit /b 1 )
"%EXE%" %*
```
Adjust `<vendor-codex-dir>` to the actual install path.

## Gemini CLI
- Install the Gemini CLI per Google's instructions.
- Auth: sign in with the Google account that holds the relevant plan (an
  interactive login draws that plan's quota; API-key auth is a separate metered
  lane — pick deliberately).
- Verify: `gemini -p "say hello" </dev/null`.

## Ollama (local models)
- Install Ollama, then `ollama pull <model>` for each model you want.
- No auth; runs offline. Verify: `ollama run <model> "hi"`.

## `llm` CLI (many providers via one tool)
- Install: `pipx install llm` (or `pip install llm`).
- Add providers via plugins, e.g. `llm install llm-openrouter llm-gemini llm-ollama`.
- Set keys in the keystore (never in files): `llm keys set <provider>`.
- Verify: `llm -m <model> "hi" </dev/null`.

## Antigravity CLI (`agy`)
- Install per the vendor's official instructions only.
- Auth: interactive sign-in on the subscription account. See the non-TTY caveat in
  `cli-invocations.md` — prefer running it interactively rather than piping.

## Billing hygiene
- Interactive/subscription login vs API-key auth can hit **different quota pools**
  even for the same account/provider. Confirm which one a CLI is drawing before
  relying on "it's free".
- Local (Ollama) is the only truly free/private lane. Everything hosted is
  metered or rate-limited — say which in the routing table.
