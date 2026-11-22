# Project Pulse Dashboard — Implementation Plan

Created by the Planner agent.

## Summary

Mona's team needs **Project Pulse**, a small static dashboard. It shows each project's name, owner, status, recent activity, and priority as polished, accessible cards. The plan covers four files and a phased handoff between the **Designer** and the **Coder**.

| File | Purpose |
| --- | --- |
| `app/index.html` | Semantic page shell and a small inline script. The script loads `project-data.json` and renders the cards. |
| `app/styles.css` | Card-based visual design with status and priority badges and a responsive grid. |
| `app/project-data.json` | Strict JSON with a top-level `projects` array. |
| `.vscode/launch.json` | Strict JSON, no comments. The **Run Project Pulse Dashboard** configuration serves `app/` on port 5500 and opens `http://localhost:%s/index.html`. |

**Repository findings that shape the plan:**

- `app/` exists but is empty. `.vscode/` only has `tasks.json`, which runs the Copilot CLI terminal when the folder opens. **Do not modify `tasks.json`.**
- The Step 3 workflow (`.github/workflows/3-step.yml`) only checks file contents for keywords. The plan must keep these literal strings:
  - `app/index.html`: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`
  - `app/styles.css`: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`
  - `app/project-data.json`: `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`, and it must pass `python3 -m json.tool`
  - `.vscode/launch.json`: `Run Project Pulse Dashboard`, `index.html`, and it must pass `python3 -m json.tool`
- **The rendering script stays inline in `app/index.html`, with no separate `app.js`.** This keeps the brief's three-file app, and the key names `status`, `recentActivity`, and `priority` then appear in `index.html` as the workflow expects.
- The Step 3 prompt fixes the launch command (`python3 -m http.server 5500`) and the serverReadyAction URL (`http://localhost:%s/index.html`).
- **No agent stages, commits, or pushes.** The learner does all git work through Copilot CLI prompts.

---

## Shared contract (freeze this before Phase B)

The Designer and Coder build against this contract so they can work on separate files at the same time. Any change to it must go through the Orchestrator.

### Data schema (`app/project-data.json`)

- **Root:** an object with one required key, `projects`, which is an array.
- **Required string fields on every project:**
  - `name`
  - `owner`
  - `status`
  - `recentActivity`: a short sentence, optionally with a date, such as "Merged accessibility fixes on Nov 18".
  - `priority`
- **Optional string field:** `summary`, one plain-language sentence for contributors. Card layout must not break when it is missing.
- **Allowed `status` values:** `On track`, `At risk`, `Blocked`, `Completed`
- **Allowed `priority` values:** `High`, `Medium`, `Low`
- **Sample data:**
  - 4–6 fictional projects.
  - Together they cover every status value and every priority value.
  - At least one project is owned by Mona.
  - At least one name or activity is long, to test text wrapping.

### Markup and class contract (`app/index.html` ↔ `app/styles.css`)

| Element | Class / attribute | Notes |
| --- | --- | --- |
| `<title>` and `<h1>` | `<h1 class="dashboard__title">` | Exact text "Project Pulse" |
| Page wrapper | `<main class="dashboard">` | Required hook `.dashboard` |
| Header | `header.dashboard__header`, `p.dashboard__subtitle` | Short description for contributors |
| Summary strip | `section.dashboard__summary` > `ul.summary-list` > `li.summary-item` (`.summary-item__value`, `.summary-item__label`) | Counts: total projects, plus counts by status |
| Message area | `p.dashboard__message` with `role="status"` and `aria-live="polite"` | Loading, error, and empty states |
| Card list | `ul.project-grid` > `li` | Responsive grid |
| Card | `article.project-card` with `data-status="{slug}"` and `data-priority="{slug}"` | Required hook `.project-card` |
| Card heading | `h2.project-card__name` | |
| Status badge | `span.status-badge.status-badge--{slug}` | Slugs: `on-track`, `at-risk`, `blocked`, `completed`, `unknown` |
| Field list | `dl.project-card__meta` > `div.project-card__field` > `dt` + `dd` | Holds Owner and Priority |
| Priority badge | `span.priority-badge.priority-badge--{slug}` | Slugs: `high`, `medium`, `low`, `unknown` |
| Activity | `div.project-card__activity` > `h3` "Recent activity" + `p.project-card__activity-text` | |
| Summary | `p.project-card__summary` | Only rendered when present |
| Footer | `footer.dashboard__footer` | Optional; could say where the data comes from |

**Slug rule:** lowercase the value, trim it, and replace spaces with hyphens. Any value not in the allowed lists becomes `unknown`.

---

## Ordered implementation steps

1. **Save this plan.** The learner or Orchestrator saves it to `docs/project-pulse-plan.md`.
2. **Write the design specification (no file edits).** The Designer returns:
   - **Visual direction:** a color palette with contrast ratios, a type scale, spacing scale, radius and shadow tokens, and badge colors for each status and priority.
   - **Layout:** grid breakpoints and the order information appears on a card.
   - **Accessibility:** requirements, plus confirmation of the markup contract or any proposed changes.

   The Orchestrator approves the spec and freezes the contract.
3. **Create the sample data.** The Coder creates `app/project-data.json` following the schema above.
4. **Create the launch configuration.** The Coder creates `.vscode/launch.json`. It is strict JSON with `"version": "0.2.0"` and one configuration with these fields:

   | Field | Value |
   | --- | --- |
   | `name` | `"Run Project Pulse Dashboard"` |
   | `type` | `"node-terminal"` (built into VS Code, so no Python extension is needed) |
   | `request` | `"launch"` |
   | `command` | `"python3 -m http.server 5500"` |
   | `cwd` | `"${workspaceFolder}/app"` |
   | `serverReadyAction.pattern` | Matches Python's "Serving HTTP on … port 5500" line, capturing the port number |
   | `serverReadyAction.uriFormat` | `"http://localhost:%s/index.html"` |
   | `serverReadyAction.action` | `"openExternally"` |

   Optional: set `"env": { "PYTHONUNBUFFERED": "1" }` so the "Serving HTTP" line is printed right away.
5. **Build the page.** The Coder creates `app/index.html` following the markup contract:
   - Use `<html lang="en">`, a viewport meta tag, and `<title>Project Pulse</title>`.
   - Link to `styles.css`.
   - The inline script:
     - Loads `project-data.json` with `fetch` relative to the page.
     - Checks that `projects` is an array.
     - Builds the cards using `textContent`, never `innerHTML` with data values.
     - Computes the summary counts.
     - Handles loading, error, and empty states in the message area.
6. **Style the dashboard.** The Designer creates `app/styles.css` from the approved spec:
   - Selectors for `.dashboard` and `.project-card`.
   - `border-radius` and `box-shadow` on cards.
   - A responsive grid using `repeat(auto-fill, minmax(…))` and/or media queries.
   - Status and priority badge modifiers, including `--unknown`.
   - Visible `:focus-visible` styles.
   - Support for `prefers-reduced-motion`.
   - Text wrapping for long content.
7. **Review the integration.**
   - The Designer reviews the rendered markup in `app/index.html` against the spec, without editing it, and reports any gaps.
   - The Coder fixes gaps in `index.html`. The Designer fixes gaps in `styles.css`.
8. **Validate.** The Coder runs the automated checks listed under Validation expectations. The learner does the manual browser check through Run and Debug.
9. **Report.** The Orchestrator summarizes which agents did what, which files changed, the validation results, and any remaining risks. This feeds `docs/final-handoff.md` in Step 4.

---

## File assignments

| File | Owner | Step | Other agents |
| --- | --- | --- | --- |
| `docs/project-pulse-plan.md` | Orchestrator / learner | 1 | Planner writes the content |
| `app/project-data.json` | **Coder** | 3 | Designer may suggest sample content |
| `.vscode/launch.json` | **Coder** | 4 | None |
| `app/index.html` | **Coder** | 5, 7 | Designer reviews only and owns accessibility and markup guidance |
| `app/styles.css` | **Designer** | 6, 7 | Coder must not edit it |
| `.vscode/tasks.json` | Nobody | None | Must stay unchanged |
| `docs/final-handoff.md` | Orchestrator (Step 4) | After 9 | Out of scope here |

Each file has exactly one owner that may write to it.

## Designer responsibilities

- Produce the design spec in step 2:
  - Visual tokens.
  - Badge color system. Every badge must also show its text, so meaning never depends on color alone.
  - Information hierarchy on each card: name, then status, then owner and priority, then recent activity, then summary.
  - Breakpoints: one column on phones, two on tablets, three or more on desktop.
- Own `app/styles.css`. Make it a polished dashboard rather than a plain page, with rounded cards, shadows, readable spacing, clear typography, and a strong header.
- Meet WCAG AA contrast (4.5:1 for body text and badge text), keep focus visible, and avoid horizontal scrolling at 320px wide.
- Confirm the markup contract in step 2 and review `app/index.html` in step 7 without editing it.
- Report design decisions, tradeoffs, files touched, and recommended visual checks.

## Coder responsibilities

- Own `app/project-data.json`, `.vscode/launch.json`, and `app/index.html`.
- Follow the markup contract exactly, so the Designer's CSS applies without changes.
- Use clear, explicit rendering logic:
  - Check the data shape.
  - Fall back to `unknown` slugs for unexpected values.
  - Show a readable error message when `fetch` fails or the JSON is invalid. Mention that the page must be served over HTTP, not opened with `file://`.
- Keep both JSON files strict: no comments, no trailing commas.
- Run the automated checks (the Coder has the `execute` tool) and report the results and any remaining risks.

---

## Dependencies

| Step | Depends on | Reason |
| --- | --- | --- |
| 2 (design spec) | 1 | Needs the plan and contract |
| 3 (data) | 1 | Needs the schema only |
| 4 (launch) | 1 | Only depends on the folder layout and file names |
| 5 (`index.html`) | 2 (frozen contract) and 3 (data schema) | Class names, accessibility guidance, and data shape must be fixed first |
| 6 (`styles.css`) | 2 (spec and contract) | Implements the approved spec |
| 7 (integration review) | 5 and 6 | Needs both files |
| 8 (validation) | 3–7 | Checks the finished result |
| 9 (report) | 8 | Reports validation results |

## Parallel work decisions

The Orchestrator should plan these phases:

| Phase | Work | Parallel or sequential | File-scope reasoning |
| --- | --- | --- | --- |
| **A** | Step 2 (Designer spec) **and** steps 3 + 4 (Coder: `app/project-data.json`, `.vscode/launch.json`) | **Parallel** | The Designer writes no files in step 2, and the Coder's two files don't depend on design decisions. No overlap. |
| **Gate** | Orchestrator approves the spec and freezes the contract | **Sequential** | Phase B needs one agreed set of class names and markup rules. |
| **B** | Step 5 (Coder: `app/index.html`) **and** step 6 (Designer: `app/styles.css`) | **Parallel** | Different files with one owner each. They connect only through the frozen contract, so neither waits for the other's output. |
| **C** | Step 7 (review, then fixes) | **Sequential** | The review needs both files. Each agent fixes only its own file. If both need fixes, they can run in parallel because the files don't overlap. |
| **D** | Step 8 (validation), then step 9 (report) | **Sequential** | Validation checks the integrated result, and the report depends on it. |

**Must not run in parallel:**

- Two agents writing to the same file.
- Any work on `index.html` or `styles.css` before the contract is frozen.
- Validation before Phase C is finished.

If the contract changes during Phase B, pause both agents and update the contract before continuing.

---

## Edge cases

- **Opened with `file://`:** `fetch` fails. Show a clear message in `.dashboard__message` explaining that the page must be run with **Run Project Pulse Dashboard**.
- **Problems with the data file:**
  - Missing file, a 404, or invalid JSON: show an error state. The page must not stay blank.
  - `projects` is missing, not an array, or empty: show "No projects to show yet".
  - Missing fields, or unknown status/priority values: show "Not provided" and use the `unknown` badge style. Never render the literal word `undefined`.
- **Script injection:** project text containing HTML must render as plain text. Use `textContent` only.
- **Long text:** long names, owners, or activity must wrap with `overflow-wrap: anywhere` so cards don't overflow.
- **Directory listing:** the launch URL must end with `/index.html`. `cwd` must be `${workspaceFolder}/app`, not the repository root.
- **Port 5500 already in use** (for example by Live Server or an earlier run that wasn't stopped): Python fails with "Address already in use". Note this in the handoff and tell the learner to stop the earlier run first.
- **Output buffering:** if the "Serving HTTP" line isn't printed promptly, the browser never opens. `node-terminal` (a terminal) and/or `PYTHONUNBUFFERED=1` prevents this.
- **Codespaces:** `localhost:5500` is forwarded automatically. `openExternally` should open the forwarded URL, but the learner may need to allow the port in the Ports panel.
- **Browser cache:** after editing the JSON, a hard refresh may be needed.
- **Accessibility:**
  - Status must not be shown by color alone.
  - Headings go h1 → h2 → h3 in order.
  - Contrast must work on the card background.
  - Respect `prefers-reduced-motion` if hover effects are used.
- **Small screens:** a single column at 320–375px with no horizontal scrolling.

## Validation expectations

**Automated, run by the Coder:**

1. `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed.
2. A small Python check confirms `projects` is a non-empty list and that every item has non-empty `name`, `owner`, `status`, `recentActivity`, and `priority` strings.
3. Keyword checks, matching `.github/workflows/3-step.yml`:
   - `app/index.html`: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`
   - `app/styles.css`: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`
   - `.vscode/launch.json`: `Run Project Pulse Dashboard`, `index.html`, `${workspaceFolder}/app`, `5500`
4. Smoke test:
   - Start `python3 -m http.server 5500` from `app/` in the background.
   - Fetch `/index.html` and confirm the response contains "Project Pulse".
   - Fetch `/project-data.json` and confirm a 200 response.
   - Stop the server.
5. `.vscode/tasks.json` is unchanged.
6. `git status` (read-only) shows the new files as untracked or modified, not staged. No git write commands are run.

**Manual, done by the learner with the Designer's guidance:**

- Run and Debug → **Run Project Pulse Dashboard** → the browser opens `/index.html` and shows the dashboard with cards, not a directory listing.
- Check:
  - Badges appear for every status and priority value.
  - The summary counts match the data.
  - The layout works at about 375px, 768px, and 1280px wide.
  - You can Tab through the page and see the focus.
  - The browser console shows no errors.
- Temporarily rename the data file (or open the page with `file://`) and confirm the error message appears.
- Stop the server before continuing.

**Note:** `scripts/validate-exercise.sh` checks the template itself, not the learner's work. Its check for tracked learner files is expected to fail after the learner commits. Don't use it as a pass/fail gate for this step.

## Open questions

1. **Owner of `index.html`:** this plan gives it to the Coder, with the Designer advising and reviewing. Does the Orchestrator prefer to have the Designer write the static markup first and the Coder add the script afterwards? That would make Phase B sequential.
2. **Inline script vs. `app/app.js`:** the plan keeps the script inline to match the brief's three files and the workflow's keyword checks on `index.html`. Is a separate JS file acceptable?
3. **`summary` field:** should the optional `summary` field be added per project for the "contributor-friendly summary" in the brief? Or is the dashboard-level summary strip enough?
4. **`recentActivity` format:** this plan uses a plain string. Is a structured `{ date, text }` object wanted? That would also need the workflow's keyword checks to still pass.
5. **Status and priority wording:** confirm `On track / At risk / Blocked / Completed` and `High / Medium / Low`, or supply Mona's team's own terms.
6. **Port 5500:** it is fixed by the Step 3 prompt. Is a fallback port needed if it's taken in the learner's environment?
7. **Out of scope unless requested:** dark mode, sorting, and filtering.
