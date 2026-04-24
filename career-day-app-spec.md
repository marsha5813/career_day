# Career Day Survey App — Build Spec

## Context

I'm doing career day at an elementary school. I'll run **four 20-minute sessions** back-to-back, each with **~50 kindergartners and first graders** (ages 5-6). I have a laptop and a screen.

My job: **VP of Data Science at Verasight** (a survey firm). Previously: **U.S. Census Bureau** and **Pew Research Center**. The whole point of the session is to show these kids what my job actually is — *asking people questions and turning the answers into pictures that teach us about the world.*

## The pedagogical idea

Most "explain your job" demos for little kids fall flat because they require abstractions kids don't have yet (numbers, percentages, proportional reasoning with bars, probability). I want to lean into what 5-6 year olds CAN do:

- Stand up / sit down in response to questions
- Look at a group of pictures and tell which kind there's more of
- Guess, then find out if they were right
- Be delighted when a grown-up is surprised by something

The session flow is a live survey. I ask a question, kids stand up if the answer applies to them, I eyeball the room, and I drag a slider to roughly match the proportion. The slider populates a visualization on the big screen — and that visualization is the "aha." We made a picture together. That's what I do at work.

A key beat is the **prediction round**: kids guess the answer to a question first, then we run the real survey. The gap between guess and reality is the whole punchline of polling. Another key beat: showing them **how grown-ups answered the same question** (real data from Pew or Verasight where I have it, or plausible made-up data if not) — so they see the same visual style applied to a different population. That's the concept of a survey in one image.

## What the app needs to do

A single-page web app I can run off my laptop (served locally, or just an `index.html` I open in a browser — whichever is simpler). It needs to work reliably four times in a row with no fiddling between sessions.

### Core interaction

- I pick or type a question (e.g., "Do you have a dog?")
- I set 2-4 answer choices, each with a label and optionally an emoji/icon (dog 🐕, cat 🐈, etc.)
- For each answer choice, I have a **slider** (0 to 100%, no number shown to the kids)
- As I drag the slider, a **visualization updates live** on screen

### The visualization

**Not a bar chart. An icon grid.** Specifically:

- A fixed grid of simple person-silhouette icons (like bathroom-sign figures — generic, not cartoony kid-specific, so the same viz works when I later show "here's what grown-ups said")
- Default grid size: **10 × 10 = 100 icons** (make this configurable — I might want 5×10 = 50 for some questions)
- Icons corresponding to the slider's proportion are **shaded in a color**; the rest are **gray/unshaded**
- **Fill order: from one side**, not shuffled. A 5-year-old should instantly see "more" vs. "less" without counting. (Optional toggle for shuffled/interspersed fill, as a stretch goal — that looks more like a real sample and could be a nice subtle visual nod, but default to fill-from-one-side.)
- **No numbers, no percentages, no axis labels.** The ratio *is* the information.
- For multi-choice questions (3-4 options), each option gets its own color, and the grid fills proportionally with all the colors. (Again, contiguous blocks, not shuffled.)

### Side-by-side comparison view

For the **prediction round** and the **grown-up comparison round**, I need to show **two icon grids side by side** with labels above them like "What you guessed" and "What's actually true" — or "Kids in this room" and "Grown-ups across America." Same grid size, same icon style, same colors. The gap between the two pictures is the whole point, so they need to be visually identical except for the fill.

I should be able to:
- Save the first grid's state (the kids' guess)
- Run the real survey
- Reveal the second grid next to the first

### Controls I need

- **Question setup:** a simple form to enter the question text and the answer choices (label + emoji + color for each). Or preset buttons for the questions I've planned.
- **Sliders:** one per answer choice. Dragging updates the grid live. For multi-choice, sliders should be constrained to sum to 100% (or auto-normalize — whichever is less fiddly in the moment).
- **Big screen mode:** a clean presentation view with just the question, the icon grid(s), and the answer labels. Controls (sliders, inputs) can be on a separate "operator" panel, or hidden behind a toggle. I'm the only one touching the laptop, so it's okay if controls are visible — but the grid and question need to be the visually dominant things on the screen.
- **Reset / next question:** clear the grid and move to the next question fast. I have 20 minutes per session and I want ~3-4 rounds per session.

### Presets (nice to have)

Pre-load the questions I'm planning to ask so I don't fumble with typing mid-session:

1. "What's your favorite ice cream?" — chocolate 🍫, vanilla 🍦, strawberry 🍓, something else ⭐
2. "Do you have a pet?" — dog 🐕, cat 🐈, another pet 🐠, no pet 🚫
3. "Which superpower would you pick?" — fly 🦅, invisible 👻, super strong 💪, read minds 🧠
4. "Would you rather…" (TBD — a fun binary)
5. A "grown-ups" comparison slot — I'll pre-enter a plausible distribution from real poll data and show it next to the kids' result.

Make the presets easy to edit in a config file or JSON at the top of the code — I'll tweak them.

## Technical preferences

- **Single HTML file** with inline CSS and vanilla JS is ideal. No build step, no npm install, no server required. I should be able to double-click the file and go.
- If a tiny framework makes it way better, Preact via CDN or similar is fine. No React build toolchain.
- Make it work offline. The school wifi will be bad.
- Icons: use inline SVG for the person silhouette (one path, embedded in the JS as a constant). Don't rely on an icon CDN.
- Colors: pick a palette that's high-contrast and friendly. The "shaded" color should really pop against the gray.
- Full-screen friendly: pressing F11 should give me a clean presentation surface. No weird overflow or tiny text.
- Font size: big. Assume the screen is across a gym/cafeteria and the kids in the back row need to read the question.

## What I do NOT want

- No numbers displayed to the audience (no "60%", no "30 out of 50")
- No bar charts (proportional reasoning on bar length is too abstract for this age)
- No animations that take more than ~300ms — kids will lose focus waiting
- No login, no accounts, no analytics, no network calls
- No dependencies that require the internet at runtime

## Build approach

Start with the simplest version that works end-to-end:

1. **v1:** One question on screen, 2 answer choices, two sliders, one 10×10 icon grid that fills from the left with two colors. Confirm the visual reads well at a distance.
2. **v2:** Add the side-by-side comparison view (two grids, labeled).
3. **v3:** Add the preset questions and a way to cycle through them.
4. **v4:** Polish — full-screen mode, big fonts, tested at actual screen sizes, smooth slider-to-grid updates.

Test by standing ~15 feet from the laptop screen and asking: *can a 5-year-old tell at a glance which color is "more"?* If yes, ship it.

## Session flow (for reference — the app supports this, doesn't enforce it)

~20 minutes, four times:

- **2 min** — intro ("my job is asking people questions and turning the answers into pictures")
- **4 min** — warm-up survey (ice cream or similar, easy and fun)
- **6 min** — prediction round (kids guess, then we survey, reveal the two grids side by side)
- **5 min** — grown-up comparison round (how did adults answer a similar question? show two grids.)
- **3 min** — wrap + buffer

## Summary for the Claude Code session

Build me a single-file HTML app that lets me, in real time in front of 50 kindergartners:

1. Display a question and 2-4 answer choices with emoji icons
2. Drag sliders (one per choice) to set proportions
3. Watch an icon grid of 100 little person silhouettes fill with colors matching those proportions, filling from one side (not shuffled)
4. Show two such grids side by side to compare "guess vs. reality" or "kids vs. grown-ups"
5. Cycle quickly through 3-4 preset questions per session
6. Look great on a big screen across a cafeteria, with no numbers or labels cluttering the visualization — the proportion of colored vs. gray icons IS the data

No build step. No network. Works offline. One `.html` file I can double-click.
