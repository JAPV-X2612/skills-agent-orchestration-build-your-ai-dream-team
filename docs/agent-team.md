# Mona's Project Pulse agent team

This team uses GitHub Copilot CLI in a Codespace to coordinate the work needed to
build Mona's Project Pulse dashboard. Each custom agent has a focused role:

| Agent | Model | Responsibility | Definition |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 4.7 | Coordinates the specialist agents, assigns non-overlapping file scopes, manages dependencies and parallel work, verifies the integrated result, and reports the final runnable dashboard. | `.github/agents/orchestrator.agent.md` |
| **Planner** | Claude Opus 4.7 | Researches the repository and requirements, then creates the implementation plan with file assignments, dependencies, edge cases, and validation expectations. | `.github/agents/planner.agent.md` |
| **Coder** | GPT-5.5 | Implements the assigned dashboard code, keeps behavior explicit and testable, creates required runnable-app support such as `.vscode/launch.json`, and validates the change. | `.github/agents/coder.agent.md` |
| **Designer** | Gemini 3.1 Pro | Defines and implements the Project Pulse user experience, including information hierarchy, accessibility, responsive behavior, visual clarity, project cards, status badges, and priority treatment. | `.github/agents/designer.agent.md` |

The Orchestrator first asks the Planner for a Project Pulse implementation plan.
It then delegates design and coding work to the Designer and Coder using clear
file ownership, runs independent work in parallel when possible, and sequences
dependent work when necessary. Finally, it verifies that the polished dashboard
and its launch configuration work together before handing the result back.
