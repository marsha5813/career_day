# Career Day Survey App

A single-file HTML app Joey runs during his elementary-school Career Day sessions. He introduces himself as a survey researcher ("my job is asking people questions and turning the answers into pictures"), then runs 3–4 live surveys per 20-minute session with ~50 kindergartners/first-graders. Kids stand up if an answer applies, Joey drags sliders to match the show-of-hands, and a 10×10 grid of emojis fills in on the projector. Key pedagogical beats: a **prediction round** (kids guess first, then reality) and a **grown-ups comparison round** (kids vs. adults on a related question).

The original full spec is in `career-day-app-spec.md`. This doc is the up-to-date operator and engineer's guide.

---

## Day-of quick start (read this at the school)

**To launch the app:**

Double-click `index.html` in Finder — it opens in your default browser. Nothing else is needed; there are no dependencies, no network calls, no login.

**If double-click doesn't work** (rare — some browser settings block local files):

```bash
cd /Users/joey/dev/career_day
python3 -m http.server 8765
```

Then open `http://127.0.0.1:8765/index.html` in your browser. Leave the terminal window open during the session.

**Once open:**

- `F11` — fullscreen (browser-native; the app is designed to look great in fullscreen on a projector)
- `P` — toggle presentation mode (hides the operator controls so only question + grids are visible to the kids)
- `Esc` — exit presentation mode
- `Space` / `PgDn` — advance to the next question card (cards scroll-snap)
- `Shift+Space` / `PgUp` — previous card
- `G` — toggle grown-ups grid visibility on the card currently in view

**Between sessions:**

Hit the **Reset All** button in the top-right of the operator bar. This clears the kids' guess and actual answers on every card but preserves the grown-ups placeholder data so you don't have to re-enter it.

---

## Files

- `/Users/joey/dev/career_day/index.html` — the entire app (HTML + CSS + JS inline)
- `/Users/joey/dev/career_day/joey.png` — portrait photo used on the intro card
- `/Users/joey/dev/career_day/career-day-app-spec.md` — original spec
- `/Users/joey/dev/career_day/CLAUDE.md` — this doc

No build step, no `npm install`, no server required at runtime. Inline CSS and inline vanilla JS, no frameworks. Emojis and math symbols (π, Σ, ∞, etc.) are native Unicode. The only local asset is `joey.png`, loaded via relative path — it works fine when opened via `file://`.

---

## How the app is structured (top to bottom of `index.html`)

1. `<style>` — CSS variables at the top (palette, sizes), then layout, then per-mode styles.
2. `<body>` — fixed operator header bar, then a `<main>` container that holds one `<section class="card">` per question. Sections are built dynamically from the `PRESETS` array when the page loads.
3. `<script>` — in top-to-bottom order:
   - `CONFIG` (grid rows/cols default)
   - `PRESETS` (the array of question cards — the main thing to edit)
   - `state` and `makeInitialState` / `resetAllState` (per-card runtime values)
   - `allocateConstrained` / `allocateSingle` (math: slider values → which cells get which color)
   - `buildGrid` (build a 10×10 DOM grid of emoji-ready cells)
   - `buildCard` (build one full card: question + slots + controls)
   - `onSliderInput` (handles slider changes, including the sum-to-100 rescaling)
   - `renderCard` (syncs DOM to state for one card — called on every slider change)
   - `INTRO` + `buildIntroCard` (the static "Hi I'm Joey" card shown first — no sliders, no state)
   - `init` (wires everything up and attaches global keyboard handlers)

---

## Where to edit common things

**Edit the intro card (photo, name, title, math decor):**

The `INTRO` object near the top of `<script>`, right above `buildIntroCard`. Fields: `photoSrc` (relative path — default `./joey.png`), `hello`, `name`, `title`, and `decor` (the row of math symbols under the name). Each decor entry has `symbol` (any Unicode character — π, Σ, ∞, √, etc.), `color` (one of `c1`..`c4` from the palette), `rotate` (degrees), and `size` (em-relative font size). Edit freely — the decor is purely visual. To swap the photo, drop a new file into the directory and update `photoSrc`.

**Change palette colors (blue / orange / green / magenta / gray):**

CSS `:root` block at the top of `<style>`. Variables `--c1` through `--c4` are the choice colors; `--c-gray` is the placeholder-circle color.

**Change, add, or reorder questions:**

The `PRESETS` array near the top of `<script>`. Each entry is a card. Fields:
- `id` — unique slug, used for DOM ids and state keys
- `question` — shown huge at the top of the card
- `choices` — array of `{label, emoji, colorIndex}`. Up to 4 choices.
- `constrainTo100Default` — `true` makes sliders rescale to sum to 100 (mutually-exclusive questions). `false` makes sliders independent (select-all questions like "do you have a dog? a cat?"). The operator can toggle this per-card live via the 🔒 / 🔓 pill.
- `grownupsData` — array of numbers per choice. These are the **PLACEHOLDER** values for the grown-ups comparison slot. Replace these with real Pew / Verasight numbers when you have them. For constrained cards, they should sum to ~100; for unconstrained, each is its own 0–100.
- `gridRows` / `gridCols` — optional per-card override of the default 10×10

**Change the default grid size:**

`CONFIG.gridRows` and `CONFIG.gridCols` at the top of the script. Default 10×10 = 100 cells.

**Change emoji size inside a cell:**

CSS rule `.cell > span { font-size: calc(75cqi / var(--cols, 10)) }`. The `75` is a percentage-of-cell — lower it for smaller emojis, raise it for larger.

**Change the placeholder dot:**

CSS rule `.cell::before`. `width`/`height` are the circle size (as a % of cell), `opacity` controls how faint.

---

## How rendering works

Each card has three **slots**: `guess`, `actual`, and `grownups`. The operator picks which slot to edit (one at a time, via the Edit pills) and which slots are visible (any combination via the Show checkboxes, including none). When the operator drags sliders, it mutates `state[cardId].values[activeSlot]`.

Two display modes per slot, driven by the 🔒 toggle:

- **Constrained (sum to 100%)**: one combined grid per slot. Cell allocation uses largest-remainder rounding so the visible cell counts always sum to exactly 100. Contiguous fill, starting with choice 0.
- **Unconstrained (independent)**: one small grid per choice per slot ("small multiples"). Each choice's slider independently controls how many cells in that choice's mini grid are filled. Used for select-all questions.

When no slots are visible (all Show boxes unticked), a dedicated placeholder-only grid appears in the slot row so the card's vertical height stays consistent. This supports Joey's "set sliders blind, then tick to reveal" workflow.

Cells never get re-created on slider change. Each cell pre-renders all four possible emoji variants plus a CSS `::before` placeholder circle; `data-color` on the cell (set by `paintGrid`) controls which variant is visible via CSS selectors. This keeps updates well under 300ms even for 12 simultaneously-visible grids.

---

## Key design decisions (and why)

- **Single HTML file, no build, no network.** Non-negotiable from the spec — must work offline on a school's flaky wifi, double-click to launch, no terminal required on the day.
- **Emoji-in-grid, no shaded silhouettes.** Earlier iteration used colored bathroom-sign silhouettes. Joey decided emojis are more engaging for this age and didn't want silhouettes at all, even as placeholders. Placeholder is now a faint circle.
- **Faint-circle placeholders (not empty cells).** Preserves the 10×10 grid structure as a visual reference so the proportion reads — "X of 100" rather than a loose cloud of emojis.
- **Slot model with up to 3 slots per card.** Lets Joey show any combination of guess / actual / grown-ups side-by-side for comparison rounds. Any Show combination is allowed, including none.
- **No guard forcing "at least one slot visible."** Joey deliberately wants blind mode for dramatic reveals — drag sliders invisibly, then tick a Show box to reveal.
- **Scroll-snap one-card-per-screen.** Keeps the presentation focused on the current question; advancing is a single keystroke with no click targets.
- **Presentation mode (`P` key) hides controls.** Same file, same window — no separate presenter window to drag to the projector.
- **Reset All preserves grown-ups data.** Grown-ups values are reference data (from Pew / Verasight), not kid-entered, so clearing them between sessions would force re-entry.

---

## Non-negotiables from the spec (don't break these)

- No numbers, no percentages, no axis labels visible to the audience. The ratio of colored cells *is* the data.
- No bar charts.
- No animations longer than ~300ms.
- Must work offline. No external resources, no CDNs, no fonts, no icon libraries.
- Must be readable from ~15 ft away on a projected screen.
- Must run reliably across four back-to-back sessions without fiddling.

---

## Keyboard shortcuts (reference)

| Key | Effect |
|---|---|
| `P` | Toggle presentation mode (hide operator controls) |
| `Esc` | Exit presentation mode |
| `F11` | Browser fullscreen |
| `Space` / `PgDn` | Next card |
| `Shift+Space` / `PgUp` | Previous card |
| `G` | Toggle grown-ups slot visibility on the current card |

---

## Troubleshooting

**The app opens but looks broken / unstyled.**
Hard-reload with Cmd+Shift+R (some browsers cache aggressively). If it still looks wrong, check the browser console for errors (Cmd+Option+J in Chrome/Safari).

**Scrolling between cards feels off.**
Make sure the browser window is in focus and there's no modal or devtools open. Scroll-snap needs mandatory snapping, which can be disrupted by overlays.

**Emoji appearance is weird.**
Emojis are native Unicode and rendered by the OS. On macOS they should look clean. If you're presenting from a different laptop, emoji font rendering may vary — verify before the session.

**Reset All doesn't clear the grown-ups column.**
That's intentional — grown-ups data is preset reference data, not kid-entered state. To reset it too, reload the page (this re-reads `PRESETS`).

**Sliders stop responding.**
Check the 🔒 pill state. In constrained mode, if all sliders are at 0 and you drag one, the remainder is distributed equally across the rest — this can look "stuck." Try unlocking (🔓) temporarily if the math feels wrong.

**The tab is open but I can't find the URL.**
If you launched via `python3 -m http.server 8765`, the URL is `http://127.0.0.1:8765/index.html`.

---

## Stopping the local HTTP server (if you started one)

If you started `python3 -m http.server` in a terminal, stop it with Ctrl+C in that terminal. If you backgrounded it, find and kill:

```bash
lsof -ti:8765 | xargs kill
```

---

## Future work / known rough edges

- Grown-ups data is all **placeholder**. Search for `PLACEHOLDER` in `index.html` to find each one; replace with real Pew / Verasight figures before the day.
- The "Would you rather" question is a placeholder (dragon vs. unicorn). Swap in whatever final question you choose.
- Small-multiples layout (unconstrained cards with 3 choices) has some vertical whitespace. Legible and presentable, but could be tightened if it bugs you.
- Controls are visible to kids in the default (non-presenting) view. Hit `P` for a clean look; there's no dual-monitor setup.
