# Agent team

I am using GitHub Copilot CLI in a Codespace to orchestrate a team of custom agents that build Mona's Project Pulse dashboard. All agent definitions live in `.github/agents/`.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 5.5 | Breaks down requests, delegates to specialists with explicit file scopes, runs work in parallel only when scopes don't overlap, verifies the integrated result. Does not implement. | `.github/agents/orchestrator.agent.md` |
| Planner | Claude Opus 5.5 | Researches the repo and docs, then returns an ordered plan with file assignments, dependencies, parallel/sequential work, edge cases, and validation. Writes no code. | `.github/agents/planner.agent.md` |
| Coder | GPT-5.5 | Implements code and fixes bugs within assigned files, plus runnable-app support such as `.vscode/launch.json` (`cwd` of `${workspaceFolder}/app`, opening `index.html`). Validates before reporting. | `.github/agents/coder.agent.md` |
| Designer | Gemini 3.7 Flash | Handles UI/UX, accessibility, and visual design: polished dashboard with project cards, status badges, priority treatment, responsive layout, and `.dashboard` / `.project-card` CSS hooks. | `.github/agents/designer.agent.md` |

## Workflow

1. The Orchestrator gets a plan from the Planner.
2. It splits the plan into phases with non-overlapping file scopes.
3. Coder and Designer implement their parts; the Orchestrator verifies and reports.

No agent stages, commits, or pushes; I control all git operations through Copilot CLI prompts.
