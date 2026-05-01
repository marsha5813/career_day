# Next Year

A scratchpad for tweaking this app the next time you run a Career Day session. Fill in the **Reflections** section while it's fresh — future-you will thank present-you.

---

## Reflections from this year

> Fill these in within a day or two while details are vivid. One-line answers are fine.

**The hits — what landed best with the kids?**
- _e.g., the prediction round on ___; the dragon vs unicorn binary; the dog bar going way past the grown-ups dog bar_

**The misses — what felt flat or confusing?**
- _e.g., kids didn't get ___; the slot switching was awkward; the fish bar reveal was too subtle_

**Timing**
- 3–4 rounds per 20-min session — did this fit? Too many? Too few?
- Which beat ate the most time? Which was rushed?

**Operational gotchas — anything you want to remember not to repeat?**
- _e.g., the projector aspect ratio was different than my laptop; emoji X rendered weirdly; F11 fullscreen toggled off when I switched tabs; ____

**What did teachers / parents / kids say afterwards?**
- _Quotes worth keeping if any._

**Real numbers — what did the kids actually answer?**
- Ice cream: _chocolate __ vanilla __ strawberry __ other __
- Pets: dog __% cat __% fish __% (across all 4 sessions)
- Superpower: _fly __ invisible __ strong __ minds __
- Would you rather (whatever the question was): _ vs _

> These are useful both for memory and for showing kids next year ("here's what last year's group said").

---

## Known to-dos for next year

These are objectively in the code right now and worth refreshing before the next event.

- [ ] **Refresh grown-ups data** for ice-cream, superpower, would-you-rather. Pets already uses real AVMA figures (43/33/3). Search `// PLACEHOLDER` in `index.html` to find each one. Pew Research does periodic polls on superpowers and food preferences — worth a search.
- [ ] **Decide on the would-you-rather question.** Currently a placeholder (dragon vs unicorn). Pick something with a clean binary that 5–6 year olds find delightful.
- [ ] **Verify the photo on the intro card is current** (`joey.png`). Or swap it.
- [ ] **Math decor on the intro card** — if you're tired of the current symbols, edit the `INTRO.decor` array. Easy fast tweak for visual freshness.

---

## Pre-event checklist

Run through this the week before:

- [ ] Open `index.html` on **the actual laptop you're bringing** to the school. Don't assume it'll work the same elsewhere.
- [ ] Connect to **the actual projector** if possible. Verify font sizes, emoji rendering, and the icon-grid legibility from ~15 ft.
- [ ] Press F11 → confirm fullscreen looks right; press P → confirm presentation mode hides controls.
- [ ] Walk through all four cards using Space / PgDn. Verify scroll-snap.
- [ ] Hit "Reset All" → verify everything clears.
- [ ] Untick all "Show" boxes → verify the placeholder dot grid appears (blind-reveal mode works).
- [ ] Open the file with **Wi-Fi off** to confirm offline behavior.
- [ ] Decide which session you'll demo presentation mode for (`P`) vs. leaving controls visible.
- [ ] If you'll do the prediction round: practice the slot switcher (Edit: guess → drag → Edit: actual → drag → Show both).

---

## Ideas to consider

Things that came up during design that you parked. Surface them again next year if useful:

- **Shuffled fill** as an option for the icon grids — would let you say "this is what a real random sample looks like." Cheap to add (the original spec mentioned it as a stretch goal).
- **Two-window mode** for projector setups where a clean presentation surface matters more than the single-keystroke `P` toggle.
- **A second "would you rather"** card if the binary format hits well — kids tend to love these and they're fast.
- **A mid-session "what do you think happens?"** beat between the warm-up and the comparison round. Could be its own card or just a riff during the prediction round.
- **Ditching the operator header** entirely if you find you never look at it during sessions. The keyboard hints are duplicated in CLAUDE.md.

---

## Things to NOT change (lessons already learned)

These were hard-won and are documented in CLAUDE.md, but worth re-flagging:

- **Don't add a bar chart with continuous lengths.** Kindergartners can't read proportional bar lengths. The 20×1 vertical-bar mode (pets) only works because cells are still discrete countable icons.
- **Don't show numbers or percentages to the audience.** The icon ratio is the data.
- **Don't add network calls.** Must run offline; school Wi-Fi is unreliable.
- **Don't introduce a build step.** This file must remain double-clickable.
