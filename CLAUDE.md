# CLAUDE.md — Morning Ritual App

## Project Overview

**"El Equilibrio del Arquitecto Visionario"** is a personalized morning ritual web application for Javier. It is a self-contained single-page application (SPA) built with vanilla HTML, CSS, and JavaScript — no frameworks, no build system, no dependencies.

The app guides users through a structured 5-phase morning routine with timed exercises, interactive inputs, and a completion summary.

---

## Repository Structure

```
morning-ritual/
└── index.html      # The entire application (HTML + CSS + JS, ~793 lines)
```

This is intentionally minimal. There is no `package.json`, no `node_modules`, no build pipeline, and no external assets. **The entire app lives in `index.html`.**

---

## Technology Stack

| Layer      | Technology                              |
|------------|-----------------------------------------|
| Markup     | HTML5 (lang="es", mobile-optimized)     |
| Styling    | Inline CSS (`<style>` tag)              |
| Logic      | Vanilla JavaScript ES6+ (`<script>` tag)|
| Frameworks | None                                    |
| Build tool | None                                    |
| Tests      | None                                    |

---

## Application Architecture

### State (lines 412–424)

All mutable state is declared at the top of the `<script>` block using plain `let` variables:

```js
let currentPhase = 0;     // index of active phase (0–4)
let currentStep = 0;      // index of active step within phase (0–1)
let timer = 0;            // seconds remaining for current step
let isActive = false;     // whether timer is running
let interval = null;      // setInterval reference for the timer
let completed = [false, false, false, false, false]; // one per phase
let intentions = {        // user-entered data, collected across phases
    vision: '',
    keyIdea: '',
    emotionalLevel: 5,
    intention: ''
};
```

### Data Model (lines 427–527)

The `phases` array defines all content. Each phase has:
- `title` — display name (Spanish)
- `icon` — emoji
- `color` — CSS class name (`phase-orange`, `phase-blue`, etc.)
- `steps[]` — array of 2 step objects

Each step object may contain:
| Field           | Purpose                                              |
|-----------------|------------------------------------------------------|
| `title`         | Step name                                            |
| `duration`      | Timer duration in seconds                            |
| `text`          | Main instructional paragraph                         |
| `guidance`      | Optional info box content                            |
| `question`      | Optional question box (blue)                         |
| `affirmation`   | Optional affirmation box (purple)                    |
| `declaration`   | Optional declaration box (gradient)                  |
| `visualization` | Optional visualization box                           |
| `scale`         | Optional scale interpretation (for range inputs)     |
| `input`         | Optional key into `intentions` object for user input |

### The 5 Phases

| # | Phase       | Color  | Icon | Duration  | Key Input          |
|---|-------------|--------|------|-----------|--------------------|
| 0 | Conexión    | Orange | ☀️   | 3+2 min   | —                  |
| 1 | Claridad    | Blue   | 🧠   | 2+3 min   | `vision` (textarea)|
| 2 | Ideas       | Yellow | 💡   | 3+2 min   | `keyIdea` (textarea)|
| 3 | Emocional   | Red    | ❤️   | 1+2 min   | `emotionalLevel` (range 1–10), `intention` (textarea)|
| 4 | Activación  | Purple | ⭐   | 1.5+1.5 min| —                |

### DOM Update Pattern

The app uses a central `updateAll()` function (line 773) that calls individual updater functions. All DOM manipulation uses `innerHTML` and `document.getElementById`. There is no virtual DOM or reactive framework.

Key functions:
| Function               | Responsibility                                  |
|------------------------|-------------------------------------------------|
| `updateAll()`          | Calls all update functions in sequence          |
| `updatePhasesGrid()`   | Rebuilds the phase navigation buttons           |
| `updatePhaseHeader()`  | Updates phase icon, title, subtitle             |
| `updateStepContent()`  | Renders step text, boxes, inputs, and timer     |
| `updateStepProgress()` | Renders the dot indicators for steps            |
| `updateProgress()`     | Updates the overall progress bar and summary    |
| `updateSummary()`      | Populates the final summary card                |
| `updateButtons()`      | Enables/disables Start and Pause buttons        |

### Timer Logic

- `startTimer()` — starts `setInterval` (1s tick), decrements `timer`
- `pauseTimer()` — clears the interval
- `resetTimer()` — pauses and resets `timer` to step's `duration`
- When `timer` reaches 0: pauses, marks `completed[currentPhase] = true`, updates UI

### Navigation

- `goToPhase(index)` — jump to any phase (resets step and timer)
- `nextStep()` — advance to next step, or next phase if on last step

### Input Handling

- `handleInputChange(field, value)` — updates `intentions[field]`
- Inputs are rendered inline within `updateStepContent()` for steps that have `step.input` defined
- The range input (`emotionalLevel`) displays a live value label; the current implementation updates `intentions` on `onchange` (not `oninput`), so the displayed value won't reflect dragging until release

---

## Styling Conventions

- All CSS is in a single `<style>` block in the `<head>`
- Uses CSS custom classes (no CSS variables or custom properties)
- Glassmorphism: `background: rgba(255,255,255,0.1); backdrop-filter: blur(16px)`
- Responsive breakpoints: `640px` (5-column phases grid) and `768px` (2-column summary grid)
- Phase colors are utility classes: `.phase-orange`, `.phase-blue`, `.phase-yellow`, `.phase-red`, `.phase-purple`
- Button colors: `.btn-green`, `.btn-yellow`, `.btn-red`, `.btn-blue`
- All colors use `rgba` or hex; no opacity filter tricks

---

## Development Workflow

### Running the App

No build step needed. Simply open `index.html` in a browser:

```bash
# Option 1: open directly
open index.html

# Option 2: serve locally
python3 -m http.server 8080
# then visit http://localhost:8080
```

### Making Changes

Since everything is in `index.html`, edits fall into three sections:

- **Lines 1–10**: `<head>` metadata (title, viewport, apple-mobile-web-app tags)
- **Lines 11–348**: CSS inside `<style>` tag
- **Lines 350–410**: HTML structure (mostly static shell; dynamic content is injected by JS)
- **Lines 411–790**: JavaScript inside `<script>` tag

When editing, be careful not to introduce markdown code fences (` ``` `) inside the HTML file — a past bug (`c094cbf`) was caused by exactly this.

### Committing

Follow the existing commit style — short, descriptive messages in English or Spanish:

```
Update index.html
Fix broken HTML: remove markdown code fences from style and script blocks
```

---

## Key Conventions

1. **No frameworks** — keep it vanilla. Do not add React, Vue, or any npm dependencies.
2. **No build tooling** — the file must remain directly openable in a browser without compilation.
3. **Single file** — all HTML, CSS, and JS stay in `index.html`. Do not split into separate files unless explicitly requested.
4. **Spanish UI** — all user-facing strings are in Spanish. Keep it that way.
5. **State is global** — all state lives at the top of the `<script>` block. No modules, no classes.
6. **DOM manipulation via innerHTML** — the app uses `innerHTML` extensively. When adding new content types, follow the existing pattern in `updateStepContent()`.
7. **No persistence** — user inputs (`intentions`) exist only in memory. There is no `localStorage`, cookies, or backend.

---

## Known Issues / Watch-outs

- **Range input display**: The `emotionalLevel` range slider updates `intentions` on `onchange`, but the displayed value in `.range-value` is static (set at render time). If you want live updates while dragging, switch to `oninput` and re-render the value label.
- **Phase completion logic**: `completed[currentPhase]` is set when the timer runs to zero, regardless of which step within the phase was active. Both steps in a phase share the same completion flag.
- **No back-navigation**: Clicking "Siguiente" is one-directional. Users can jump phases via the grid, but cannot go back within a phase's steps.
- **No state persistence**: Refreshing the page resets all progress and inputs.

---

## Branch Information

- Primary development branch: `claude/add-claude-documentation-1PpsK`
- Main branch: `main` (remote), `master` (local)
- Remote: `http://local_proxy@127.0.0.1:54192/git/jalfam/morning-ritual`
