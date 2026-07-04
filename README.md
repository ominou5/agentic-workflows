# Agentic Workflows

A collection of all my agentic workflows, skills, prompts, etc.

## 🛠️ Skills Included

* **[autodev](Skills/autodev/SKILL.md)**: Universal Autonomous Feature Development Protocol. An automated build-evaluate-iterate loop supporting:
  * **Standard Mode**: Isolated development in background Git worktrees across a dual-session architecture.
  * **Yolo Mode**: Immediate inline development branching directly inside the active workspace and terminal.
  * **Self-Customization Startup**: Auto-configures compilation commands, risk levels, and protected files during first-run initialization.
  * **CI/CD Remote Feedback**: Polls remote PR checks (e.g., GitHub Actions), parses failure logs, and patches code autonomously.

* **[delegate](Skills/delegate/SKILL.md)**: Multi-Model Orchestration & Delegation. Turns Claude Code into an orchestrator that routes each task to the right model — "right model for the right job" — and reports results back. Supporting:
  * **Self-Discovery Startup**: Detects the model CLIs (`codex`, `gemini`, `ollama`, `llm`, ...) and Claude models available on *your* machine, then builds a routing config calibrated to your plans and priorities — not a hardcoded roster.
  * **Opt-in Subagents**: Ships wrapper subagents that delegate to external models (GPT/Codex, Gemini, OpenRouter, local Ollama, etc.) and only fire on explicit request — safe for governed/brownfield repos.
  * **Hardened Delegation**: Verified CLI invocations with the stdin-hang, non-TTY stdout, versioned-model-pinning, and sandbox-isolation gotchas already solved.
