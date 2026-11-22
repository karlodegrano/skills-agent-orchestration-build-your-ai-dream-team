# Project Pulse final handoff

Mona's Project Pulse dashboard is built as a small static app. It was produced with GitHub Copilot CLI in a Codespace, using the custom agents defined in `.github/agents/`.

## Agent team

| Agent | Contribution |
| --- | --- |
| Orchestrator | Coordinated the phases, froze the shared markup contract, gave each agent an explicit file scope, and integrated the result. |
| Planner | Researched the repo and wrote `docs/project-pulse-plan.md` (file ownership, dependencies, parallel work decisions, validation expectations). |
| Designer | Wrote the design spec (palette, type, spacing, badges, breakpoints, accessibility) and owns `app/styles.css`. |
| Coder | Implemented `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`. |

Agent definitions are documented in `docs/agent-team.md`.

## What was built

| File | Owner | Purpose |
| --- | --- | --- |
| `app/index.html` | Coder | Page shell titled "Project Pulse"; an inline script fetches `project-data.json` and renders one `project-card` per project, plus a summary strip. |
| `app/styles.css` | Designer | Polished layout: indigo hero header, rounded cards with shadows, status and priority badges, and a 1/2/3-column responsive grid. |
| `app/project-data.json` | Coder | Top-level `projects` array with 5 sample projects (`name`, `owner`, `status`, `recentActivity`, `priority`, optional `summary`). |
| `.vscode/launch.json` | Coder | Strict JSON launch configuration named "Run Project Pulse Dashboard". |

## How to run the dashboard

1. Open **Run and Debug** in VS Code.
2. Select **"Run Project Pulse Dashboard"** and start it.
3. The configuration runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. When the server prints its "Serving HTTP" line, `serverReadyAction` opens `http://localhost:%s/index.html`, so the dashboard opens, not a directory listing.
4. Stop the debug session when finished.

## Dashboard validation results

Checked in this review:

- **JSON:** `app/project-data.json` and `.vscode/launch.json` both pass `python3 -m json.tool`. `launch.json` contains no comments.
- **Required content:** `app/index.html` has the title "Project Pulse", references `styles.css` and `project-data.json`, and contains `project-card`, `status`, `recentActivity`, and `priority`. `app/styles.css` has `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`.
- **Rendering (jsdom, not a real browser):** running the page script against the real files produced 5 cards with the correct status and priority badges and recent activity. The summary strip showed 5 total, 2 on track, 1 at risk, 1 blocked, 1 completed, and no literal `undefined` appeared in the page content. A failed fetch showed the message telling the user to serve over HTTP instead of using `file://`.
- **HTTP smoke test:** serving `app/` returned 200 for `index.html`, `styles.css`, and `project-data.json`, and `index.html` contained `<title>Project Pulse</title>`. Python's startup line matched the `serverReadyAction` pattern and captured the port. The test server was stopped afterwards.
- **Contrast (computed):** body text 16.3:1, muted text 7.6:1, hero subtitle 7.7:1, status badges 8.1–8.5:1, priority badges 6.5–7.5:1, accent 7.9:1. All pass WCAG AA (4.5:1).
- **Untouched:** `.vscode/tasks.json` was not modified.

Not verified:

- **Real browser:** the page was not opened in a browser, so visual appearance and the 375px, 768px, and 1280px layouts have not been seen.
- **Launch configuration:** it was not started through Run and Debug, so VS Code opening the browser has not been seen.
- **Keyboard and motion:** keyboard focus and reduced-motion behavior were not exercised.

## Remaining risks and handoff notes

- **Run once manually:** start "Run Project Pulse Dashboard" and check the layout at the three widths above, tab through the page, and confirm the browser console shows no errors.
- **Port 5500:** if another process, such as an earlier run or Live Server, is using it, Python fails with "Address already in use". Stop that process first.
- **Codespaces:** the forwarded port may need to be allowed in the Ports panel.
- **Heading order:** each card name is an `h2`, which sits at the same level as the visually hidden "Project summary" `h2`.
- **Out of scope:** dark mode, sorting, and filtering.
- **Git:** no agent staged, committed, or pushed anything. Git operations were run by the learner through Copilot CLI prompts.
