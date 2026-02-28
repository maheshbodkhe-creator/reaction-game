# React/or — Reaction Timer

A polished, single-file reaction timer game built with vanilla HTML, CSS, and JavaScript, with a Three.js particle system as the visual backdrop. Designed as a portfolio artifact demonstrating clean state management, high-resolution timing, and production-quality UI.

---

## Demo

Open `reaction-timer.html` directly in any modern browser — no build step, no dependencies to install.

---

## Features

- **Millisecond-accurate timing** using `performance.now()` for sub-millisecond resolution, immune to system clock drift
- **Randomised stimulus delay** (1.5–4.5 seconds) to prevent rhythm-based anticipation
- **5-round game loop** with per-round results and a final summary screen
- **Six distinct game states**: `intro → idle → waiting → go → result → done` (plus `early` for false starts)
- **First-load instructions** displayed inside the arena — shown once, never again after dismissed
- **Reaction time ratings** with benchmarks based on published human reaction time data:
  - 🟢 **Amazing** — under 200 ms (elite / top 5%)
  - 🟩 **Very Good** — under 250 ms (above average)
  - 🟡 **Average** — under 350 ms (typical adult)
  - 🔴 **Below Average** — 350 ms and above
- **Three.js particle system** — 200 particles orbit inside the arena and smoothly lerp between visual states (calm idle drift, agitated waiting, burst on go, red scatter on early click)
- **Spacebar support** — triggers every primary action globally, with `preventDefault` to block page scroll
- **Round history panel** with proportional bar charts, Best and Slow badges
- **Fully responsive** — works on desktop and mobile
- **Single HTML file** — all CSS, JS, and markup in one self-contained file

---

## Tech Stack

| Layer | Choice | Reason |
|---|---|---|
| Language | Vanilla JS (ES2020+) | No framework overhead; demonstrates JS fundamentals clearly |
| 3D / WebGL | Three.js r128 (CDN) | Particle system with `BufferGeometry` for zero-GC animation |
| Fonts | Google Fonts (Oxanium + Inter) | Oxanium for numeric display, Inter for UI body text |
| Styling | Plain CSS with custom properties | Design token system, state-driven attribute selectors |
| Timing | `performance.now()` | DOMHighResTimeStamp — microsecond resolution, no clock drift |

---

## Project Structure

```
/
├── reaction-timer.html   # The entire application
└── favicon.png           # Favicon (referenced from root)
```

Everything lives in a single HTML file with clearly separated sections:

```
<head>
  Design tokens, fonts, Three.js CDN import
  
<style>
  CSS custom properties → layout → arena states → component styles

<body>
  Header / score strip / arena / history panel / CTA button

<script>
  Constants & rating scale
  Three.js particle system (setup → per-frame animation loop)
  Game state object
  render() — single source of truth for all UI updates
  Game logic (startRound, handleArenaClick, resetGame)
  Event listeners (click, keydown, spacebar)
```

---

## Game States

The arena's `data-state` attribute drives all CSS transitions and Three.js particle behaviour:

| State | Trigger | Visual |
|---|---|---|
| `intro` | Page load (first visit only) | Teal glow, How to Play panel + score guide |
| `idle` | Intro dismissed / game reset | Calm, dim particles |
| `waiting` | Round started | Pulsing border, faster particles |
| `go` | Random delay elapsed | Green burst, particles scatter outward |
| `early` | Clicked before stimulus | Red particles, shake animation |
| `result` | Reaction recorded | Settled glow, time + rating chip |
| `done` | All 5 rounds complete | Summary grid with average rating |

---

## Key Technical Decisions

### High-resolution timing
`performance.now()` is captured at the exact moment the stimulus renders — not inside a debounced event handler — eliminating any scheduler jitter from the measurement. The result is `Math.round()`-ed to a clean integer millisecond.

```js
// Capture when stimulus appears:
state.stimulusAt = performance.now();

// Capture on click:
const elapsed = Math.round(performance.now() - state.stimulusAt);
```

### Anti-anticipation delay
The pre-stimulus delay is drawn from a uniform random distribution between 1500 ms and 4500 ms. The wide, unpredictable range makes it impossible to time a click based on rhythm or counting.

### State machine via `data-state`
All visual logic is driven by a single `data-state` attribute on the arena element. CSS selectors like `.arena[data-state="go"]` handle every visual variant — no class toggling soup. The `render()` function is the only place state is written to the DOM, keeping UI and logic cleanly separated.

### Three.js particle system
- `WebGLRenderer` with `alpha: true` renders to a transparent canvas layered behind the UI
- `OrthographicCamera` with a `[-aspect, aspect] × [-1, 1]` frustum maps particle positions to viewport fractions, updating on `ResizeObserver`
- `BufferGeometry` + `Float32Array` updates particle positions in-place each frame — no object allocation in the animation loop
- `AdditiveBlending` makes overlapping particles bloom, creating a natural glow
- Visual properties (colour, size, speed, spread) are stored as lerp targets; each frame smoothly interpolates `current → target` for fluid state transitions without a tween library

### Intro panel (shown once)
The intro screen is a dedicated `intro` game phase. An `introSeen` flag ensures that once dismissed, `resetGame()` always returns to `idle` — never back to the intro. Clearing history after the intro has been seen also lands on `idle`.

---

## Controls

| Input | Action |
|---|---|
| `Space` | Start game / advance round / register reaction / restart |
| Click / Tap | Same as Space, but only within the arena |
| Start Game button | Begins a new game |
| Play Again button | Resets and starts immediately |
| Clear button | Resets the game and clears history |

---

## Rating Scale Reference

Reaction time benchmarks based on published human psychomotor research:

| Rating | Threshold | Context |
|---|---|---|
| Amazing | < 200 ms | Competitive athletes, top ~5% of adults |
| Very Good | < 250 ms | Above average, trained response |
| Average | < 350 ms | Typical healthy adult |
| Below Average | ≥ 350 ms | Fatigue, distraction, or slower motor response |

> Note: these thresholds apply to **visual** reaction (click on colour change). Audio reaction times are typically 20–40 ms faster.

---

## Browser Support

Works in any browser with WebGL and ES2020 support:

- Chrome / Edge 90+
- Firefox 88+
- Safari 14+

---

## Running Locally

No server required for most browsers. Just open the file:

```bash
open reaction-timer.html       # macOS
start reaction-timer.html      # Windows
xdg-open reaction-timer.html   # Linux
```

If your browser blocks local file resources (rare), serve with any static server:

```bash
npx serve .
# or
python -m http.server 8080
```

Then visit `http://localhost:8080/reaction-timer.html`.

---

## License

MIT — use, fork, and modify freely.