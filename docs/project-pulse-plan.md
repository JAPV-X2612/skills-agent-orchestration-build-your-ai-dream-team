# Project Pulse implementation plan

## Summary and goal

Build Mona's lightweight, static **Project Pulse** dashboard for contributors.
The first view must make it easy to answer which projects are active, who owns
them, what their current status is, what happened recently, and which work is
highest priority or at risk. The result must feel like a polished frontend
dashboard rather than a bare HTML page or a server directory listing.

The implementation has four deliberately non-overlapping file assignments:

| File | Owner | Responsibility |
| --- | --- | --- |
| `app/index.html` | Coder | Semantic page shell, exact `Project Pulse` title/heading, stylesheet and JSON references, and rendering of project cards and their fields |
| `app/styles.css` | Designer | Visual system, layout, responsive behavior, accessibility states, status/priority treatment, and polished card styling |
| `app/project-data.json` | Coder | Deterministic project content in a top-level `projects` array |
| `.vscode/launch.json` | Coder | Strict JSON launch configuration for serving and opening the dashboard |

No other implementation files are needed. The Orchestrator owns coordination
and final integration review; it should not take ownership of these files.

## Requirements and content contract

### Dashboard UI

`app/index.html` must:

1. Set the document title and a prominent page heading to the exact title
   **Project Pulse**.
2. Link to `styles.css` and load/reference `project-data.json`.
3. Present a clear dashboard overview and a responsive collection of visible
   project cards. Use the deterministic `.dashboard` hook for the primary
   dashboard region and the `.project-card` hook on every card.
4. Show each project's name, owner, status, recent activity, and priority.
   Status and priority must have readable labels and accessible contrast, not
   rely on color alone.
5. Give contributors a short, plain-language summary for each project. A
   `summary` field may be included in the JSON in addition to the required
   fields; otherwise the page must provide an equivalent contributor-friendly
   sentence from the project data.
6. Use semantic headings, lists/regions as appropriate, meaningful labels, and
   a useful empty/error state if JSON loading fails. Avoid a raw directory
   listing as the user-facing experience.

### Data

`app/project-data.json` must be valid JSON with a top-level `"projects"` array.
Every project object must include:

- `name`
- `owner`
- `status`
- `recentActivity`
- `priority`

Use deterministic, realistic records so the cards demonstrate the full UI:

- **Contributor Portal** — owner **Avery Chen**, status **Active**, recent
  activity describing the contributor onboarding flow being updated, priority
  **High**.
- **Docs Refresh** — owner **Jordan Lee**, status **In progress**, recent
  activity describing the navigation and setup guide review, priority
  **Medium**.
- **Release Readiness** — owner **Sam Rivera**, status **At risk**, recent
  activity describing a pending dependency or verification task, priority
  **High**.

Each record should also include a concise `summary` if that helps the UI
communicate the contributor action or context. Keep values short enough for a
card and ensure every value displayed by the page comes from the JSON rather
than duplicated hard-coded project content.

### Visual and responsive design

The Designer must produce a polished visual hierarchy with:

- a clear page header and scannable metadata;
- a responsive grid that works on narrow mobile screens and wider desktop
  screens;
- readable spacing and typography;
- distinct status badges and priority/risk treatment with text labels;
- rounded cards using `border-radius`, subtle depth using `box-shadow`, and
  visible hover/focus states;
- sufficient color contrast, keyboard-visible focus, sensible reduced-motion
  behavior, and no information conveyed by color alone.

The stylesheet must contain the exact `.dashboard` and `.project-card`
selectors. The design should remain useful with long owner names, activity
text, or summaries and should not overflow horizontally.

## Ordered implementation phases

### Phase 1: Establish the contract

**Owner:** Orchestrator, with Coder and Designer acknowledgement.

Confirm the four file assignments, the JSON schema, the exact title
`Project Pulse`, the three representative project records, and the launch
requirements below. The Orchestrator should pass this plan to both specialists
so neither invents a conflicting schema or takes another agent's files.

### Phase 2: Parallel specialist implementation

After the contract is agreed, run these tasks in parallel because their file
scopes do not overlap:

* **Designer — `app/styles.css` only:** implement the visual hierarchy,
  responsive `.dashboard` layout, `.project-card` styling, badges, priority
  treatment, focus states, and accessibility/responsive rules described above.
  The Designer should report the class names and expected markup states to the
  Orchestrator, but must not edit the Coder-owned files.
* **Coder — `app/project-data.json`, `app/index.html`, and
  `.vscode/launch.json` only:** create the data contract and semantic page,
  render cards from the JSON, include all required visible fields and
  contributor-friendly summaries, and create the launch configuration.

The Coder may implement data loading and rendering with a small deterministic
vanilla JavaScript module/script in `app/index.html`; do not add another file
unless the Orchestrator explicitly revises the assignment. The Coder should
use safe DOM text rendering and provide a clear data-load error state.

### Phase 3: Sequential integration review

Once both specialists report completion, the Orchestrator reviews the
integrated result sequentially. First confirm that the HTML class names and
state attributes match the Designer's CSS, then confirm that every required
JSON field is rendered, and finally inspect the launch configuration against
the runtime behavior. Resolve any mismatch by assigning the smallest possible
change to the file's owner; do not have both agents edit the same file.

## Launch configuration

The Coder must create `.vscode/launch.json` as strict JSON with no comments.
It must contain a configuration with the exact name **Run Project Pulse
Dashboard**, use the command **`python3 -m http.server 5500`**, and set
`cwd` to **`${workspaceFolder}/app`**. Configure its server-ready browser action
to open **`http://localhost:%s/index.html`** (or the equivalent VS Code
`serverReadyAction` URI format), so it opens `index.html` rather than the
directory root. The configuration must serve from `app/`, use port `5500`,
and make the Project Pulse UI the first browser view.

## Dependencies and ordering decisions

* The JSON schema and content contract are a prerequisite for HTML rendering;
  therefore Phase 1 precedes implementation.
* The Designer's CSS and the Coder's HTML depend on the agreed class contract,
  but neither needs the other's file to begin. They can run in parallel after
  Phase 1.
* `app/index.html` depends on `app/project-data.json`'s field names and on the
  Designer's documented hooks, so integration validation must be sequential
  after both parallel tasks finish.
* `.vscode/launch.json` is independent of styling and data implementation and
  can be created in the Coder's parallel task, but its end-to-end browser
  behavior must be checked only after `app/index.html` exists.
* No parallel edits are allowed within one file. The Orchestrator must
  serialize fixes when a review finds an ownership conflict.

## Concrete validation expectations

### Static validation

The responsible agent must report:

1. All four assigned files exist and no unrelated files were changed.
2. `python3 -m json.tool app/project-data.json` succeeds and the parsed value
   has a top-level `projects` array; every project has `name`, `owner`, `status`,
   `recentActivity`, and `priority`.
3. `python3 -m json.tool .vscode/launch.json` succeeds, with no comments, and
   includes the exact launch name, command, `cwd`, port, and
   `index.html` URL requirements.
4. `app/index.html` contains the exact `Project Pulse` title/heading, links
   `styles.css`, references `project-data.json`, contains `.dashboard` and
   `.project-card` usage, and renders status, recentActivity, priority, owner,
   names, and summaries.
5. `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`,
   and `box-shadow`, plus responsive layout and visible focus/accessible
   states.
6. Review the generated HTML/CSS for malformed markup, missing asset paths,
   horizontal overflow at mobile widths, unreadable badge contrast, and
   missing/failed JSON handling.

### Runtime validation

From the repository root, start the configured **Run Project Pulse Dashboard**
launch target. Confirm that `python3 -m http.server 5500` runs with
`${workspaceFolder}/app` as its working directory and that the browser opens
`http://localhost:%s/index.html` (with `%s` replaced by the server port), not a
directory listing. Confirm the page visibly shows the Project Pulse heading and
all three project cards, with names, owners, statuses, recent activity,
priorities, and contributor-friendly summaries. Check keyboard focus, narrow
viewport responsiveness, and the data-load error state before stopping the
preview server.

## Risks and edge cases

- A page opened directly from `file://` may block `fetch`; validation must use
  the configured HTTP server.
- Missing, malformed, or empty JSON should produce an informative in-page
  message rather than a blank dashboard or uncaught error.
- Long activity and summary text must wrap without breaking the card grid.
- Status and priority colors must remain understandable in grayscale,
  high-contrast settings, and for color-vision differences.
- A server-ready pattern that opens `/` instead of `/index.html` is incorrect
  even if the server itself starts successfully.

## Handoff

Each specialist reports files touched, decisions, validation performed, and
remaining risks to the Orchestrator. The Orchestrator performs the final
cross-file review and reports the runnable dashboard. No agent stages, commits,
or pushes changes; git operations remain under the learner's control.
