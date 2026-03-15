# Signal2Noise AudioViz — Teaching Notes
## Prototype-to-Production: Noteworthy Prompts

This document captures the moments in the development conversation that best illustrate
the prototype-to-production discipline. Each prompt below represents a decision, instinct,
or question that pushed the project from "it works on my machine" toward something shippable.

---

### 1. "Can you validate the 'design the preset' feature? Simulate a MP3 being uploaded and ensure the preset is valid?"

**Why it matters:** This is the archetype of *validate before you ship*. The feature looked
like it worked — the UI responded, patterns appeared. But rather than ship on appearance,
the ask was for proof. We simulated the full preset-building logic in Node.js with synthetic
audio data and found 3 real bugs:

- ZCR denominator shrinking near end-of-buffer (misclassifying kicks as hi-hats)
- Hi-hat fill double-assigning onset steps
- Onset detection silently dropping consecutive hits at similar energy

No browser. No UI. Pure logic validation. This is what unit testing looks like before
unit tests exist — and it's the instinct that creates them.

---

### 2. "Expected / Actual / Wanted" (the preset switching bug report)

**Why it matters:** Structured bug reporting is a production engineering discipline.
Most people say "it's broken." This format separates what was promised, what happened,
and what success looks like — which is enough signal to trace root causes systematically.

From one structured report we found 5 distinct root causes in a single pass:
- Stale scheduler `curStep` and `nextTime` on preset switch
- Stacked `setInterval` timers (no clear before restart)
- `AudioContext` never resumed on preset click
- `buildGrid()` DOM rebuild racing with pending `markStep` `setTimeout` callbacks
- Stale `vizLevels` bleeding channel glow across preset switches

The structured format is what made all 5 findable at once rather than one at a time.

---

### 3. "Can you validate the visualizer, maybe at the 'I can get the numbers to render' level and not with the renderer unless it's easy"

**Why it matters:** Systems thinking applied to debugging. The rendering layer (Canvas,
WebAudio, rAF loops) is an expensive place to debug data problems. Recognizing that the
bug was in the data pipeline — not the renderer — meant we could trace it in Node.js
in under a minute.

We found the `b+=3` stride bug (skipping 67% of frequency bins, making kick and snare
invisible to the fire engine) and the analyser smoothing problem without touching the
canvas at all. The visualizer was a symptom. The spectrum was the disease.

---

### 4. "How about the underlying architecture of the code you created? Are there any refactoring opportunities, or missing tests? Can you identify any design patterns (think Gang of Four) which could help simplify the code?"

**Why it matters:** This is the *checkpoint before refactor* discipline — arguably the
most important moment in the entire project. The feature set was complete. The bugs were
fixed. The temptation was to keep adding. Instead: stop, zoom out, audit.

The result was a full architectural review that identified:
- 10 subsystems, 21 global mutable variables, god-object script architecture
- 5 SOLID violations (3 outright failures)
- 8 GoF pattern opportunities, prioritized by leverage
- 22 specific missing tests, categorized by type

This document became the repo's architecture review — the explicit before-state that v9
will be measured against. Without this checkpoint, refactoring is just rewriting.

---

### 5. "Should we check it in broken?" (implied: fix first)

**Why it matters:** The fact that the question was asked at all shows the right instinct.
The answer cemented the rule: **never check in broken code, especially a baseline commit.**

A broken baseline commit makes the diff story unreadable. If v8 is broken and v9 is the
"fix + refactor," you lose the ability to show what the refactor actually changed. We found
3 pre-checkin bugs by auditing the renamed file before a single byte touched the repo:

- `Object.freeze()` called on a `Float32Array` (not freezable — throws in strict mode)
- `micFreqData` variable used but never declared (dropped during rename)
- `activeHeat` declared after its first call site

The commit message for a clean baseline is a teaching artifact in itself.

---

### 6. "Since Cthugha really isn't my IP, let's rename this to Signal2Noise_AudioViz — I checked what a good name could be and Signal2Noise is used in many places, so it was free to use"

**Why it matters:** IP due diligence before publishing. This shows two good instincts
working together: recognizing that the original inspiration (Cthugha, 1993) had its own
identity worth respecting, and then *actually checking* whether the new name was clear
before committing to it. Renaming before the first public commit means there is no awkward
history to explain, no pull request titled "remove IP violation," and no confusion in the
git log. The right time to check is before the repo exists publicly — not after.

---

### 7. "I would like to save the current functional state before the refactor. I would like to also store the review you've done, such that we can use it as a comparison with what will be done."

**Why it matters:** Version control as narrative. The insight here goes beyond backup — it
is a deliberate act of creating a *before* so the *after* has meaning. The diff between
v8 and v9 is the teaching material. Anyone can clone this repo and run:

```
git diff v8-baseline..v9-refactor -- v8/signal2noise_v8.html
```

...and see exactly what Strategy, Observer, and Template Method patterns replaced, line
by line. That diff is worth more than any explanation of those patterns, because it shows
them applied to real code solving a real problem. The architecture review stored alongside
the baseline is the rubric — it tells you what the diff should accomplish before v9
is written.

---

### 8. "I would also like to be able to sample the incoming audio and build a preset"

**Why it matters:** Requirements that stay connected to real utility. This prompt came
after the sequencer was already working well — the easy move would have been to keep
polishing the UI or adding more VFX types. Instead it pushed toward something genuinely
useful: record real drums, analyze the audio, auto-classify hits into channels.

That requirement forced the spectral centroid classifier, which was then validated with
Node.js simulation, which then revealed the ZCR denominator bug, which became one of
the documented pre-production fixes. Good requirements pull complexity into the open.
Features that exist just to exist don't.

---

## The meta-lesson

Reading these prompts together, the pattern is consistent: **questions about correctness
came before questions about completeness.** The project grew in capability, but each growth
phase was followed by a pause to check whether what existed actually worked, was
architecturally sound, and was properly owned.

That sequence — build, validate, audit, document, commit — is what separates a prototype
that teaches from a prototype that just runs.

---

## v9 contract

The architecture review (`architecture/architecture_review.html`) identifies 8 GoF patterns
and 22 missing tests. v9 should be measured against those specifically. A refactor that
doesn't close the identified gaps is not a refactor — it's a rewrite with a new set of
unknown gaps.

The suggested commit sequence for v9:

1. `refactor: Strategy pattern — synth map + VFX update map`
2. `refactor: Observer/EventBus — decouple scheduler from visual subsystems`
3. `refactor: Template Method — unified VFX lifecycle loop`
4. `test: add 22 identified missing tests`
5. `refactor: Command pattern — preset actions + undo stack`
6. `refactor: Facade — injectable interfaces enabling unit test runs`

Each commit should reference the architecture review finding it closes.
