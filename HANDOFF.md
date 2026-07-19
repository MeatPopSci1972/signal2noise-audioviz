# HANDOFF — Signal2Noise AudioViz

> **First action for any fresh session (human or AI): read this file top to bottom.**
> This is a static GitHub Pages repo — no dev server, so there is no `GET /HANDOFF.md`
> or `GET /status`; this file *is* the status endpoint.

---

## 1. Current state

- **Current version:** `v9/signal2noise_v9.html` — v9 step 1 (renderer extraction)
- **Frozen baseline:** `v8/signal2noise_v8.html` — do not modify; it exists so `git diff` tells the refactor story
- **Behavior contract for step 1:** visually and audibly identical to v8
- **Validation done:** headless Node.js simulation (stubbed DOM/Canvas/WebAudio) — load+init, dispose→init lifecycle, dispatch→bus→onTrigger→tick full frame, all 6 VFX paths incl. balloon-pop over 120 frames, and exactly **1** trigger listener on the bus after repeated view switches (no leak)
- **Checkpoint status:** ⬜ human side-by-side verification v8 vs v9 pending (see §5 backlog, item 1)

## 2. File map

```
README.md                        project front door; history table; roadmaps
HANDOFF.md                       this file — state, contract, workflow, backlog
index.html                       redirect to the current version (still → v8; update after checkpoint)
v8/signal2noise_v8.html          frozen pre-refactor baseline
v9/signal2noise_v9.html          current working file (single-file, zero-dependency)
architecture/architecture_review.html   v8 rubric: GoF / SOLID / 22 missing tests
```

Everything ships as single-file HTML. No build step. proto2prod rules apply:
validate before optimize, surgical edits, visible errors over graceful degradation.

## 3. Architecture contract (v9)

### Renderer contract — every view implements exactly this

```
init(env)       env = {ctx, vx, FW, FH}. Build ALL internal state fresh.
onTrigger(e)    e = {ci, cx, xFrac, width, intensity, vol, col, vfxSet, t}
                One sequencer hit, timing already compensated. React or ignore.
tick(frame)     frame = {seqFreq, micLevel, t}. Draw exactly one frame.
dispose()       Release everything. Manager may init a different renderer
                on the same canvases immediately after.
```

### Load-bearing boundaries

- **Renderers NEVER touch WebAudio nodes.** The app loop reads analysers once
  per frame and passes data in via `frame`. This is the wall all views live behind.
- **The scheduler knows no VFX by name.** It calls `dispatchTrigger(ci, delay)`,
  which bumps the vizBar and emits `trigger` on the EventBus. `trigViz()` is
  deleted; do not reintroduce direct scheduler→visual calls.
- **App-level (shared by all views):** transport, presets, grid, vizBar,
  sampler/mic, `vizIntensity`, `CH_HEAT`, `CHANNELS` (read-only to renderers).
- **View-level (private to a renderer's closure):** every pixel buffer,
  particle pool, and animation state. Alpha holds the fire field, VFX pool,
  balloons, creatures, mountain.
- **`vfxSet` in the trigger payload is Alpha-specific** (fire/strobe/balloon/
  sparkle/firework/creature pills). Future views may ignore it. If a second
  view ever needs pills, generalize then — not before.
- **Adding a view = one factory + one `VIEWS` entry.** Nothing else changes.
- **Lifecycle self-test:** clicking the ACTIVE view tab re-runs dispose→init.
  Use it after any renderer change.

### Decisions on record

- FM synthesis will live in a **separate synth panel** (new tonal channels
  alongside drums), not per-drum-channel dials. Requires synth-side Strategy first.
- vizBar stays app-level so every view keeps the per-channel meters.

## 4. Workflow / gate

1. Make the change in `v9/signal2noise_v9.html` (or a new `vN/` when a version freezes).
2. **Gate:** syntax check + headless Node simulation of the touched seams
   (lifecycle, bus wiring, full-frame tick). No browser-only "looks fine" ships.
3. Human checkpoint in the browser — side-by-side with the previous state
   when the contract is "behavior identical."
4. Commit with a message referencing the architecture-review finding it closes,
   e.g. `refactor: Strategy + EventBus — renderer extraction, view tab bar (v9 step 1)`.
5. Never check in broken code — a broken baseline makes the diff story unreadable.
6. When a version is complete: update `index.html` redirect, README history table,
   and this file's §1/§5, then tag.

## 5. Backlog (ordered)

1. ⬜ **Checkpoint v9 step 1** — v8 vs v9 side by side; preset switch; mic record;
   click active tab mid-playback (fire resets, music never stutters)
2. ⬜ Commit step 1; update `index.html` redirect → v9
3. ⬜ **Spiral view** — feedback buffer + pixel ghosting (translucent fade + rotate,
   never clear). Smallest view; stress-tests `dispose()`. Checkpoint: switch
   Alpha ↔ Spiral repeatedly, zero ghost state.
4. ⬜ Sand view — particle deposition, per-channel pour points, heightmap + angle of repose
5. ⬜ Organic view — metaballs via radial gradients + threshold pass
6. ⬜ Galactic view — softened n-body, triggers add mass/velocity
7. ⬜ Synth-side Strategy — `synth()` switch → voice map (arch review finding)
8. ⬜ FM voices + synth panel — 2-op FM (mod → gain(index) → carrier.freq),
   dials: ratio / index / decay
9. ⬜ Write the 22 missing tests (arch review) on the Node-simulation scaffolding
10. ⬜ Command pattern — preset actions + undo stack
11. ⬜ State machines — sequencer + sampler

## 6. Development discipline (distilled from the v1–v9 history)

The full prompt-by-prompt narrative lived in TEACHING_NOTES.md (see git history,
pre-v9-step-1). The load-bearing lessons:

- **Validate before you ship.** Simulate the logic in Node.js with synthetic
  data before trusting the UI. This caught the ZCR denominator bug, the hi-hat
  double-assign, dropped consecutive onsets (v5); five scheduler/state bugs
  from one structured report (v6); the `b+=3` analyser stride bug (v7); and
  three pre-baseline bugs including `Object.freeze` on a Float32Array (v8).
- **Expected / Actual / Wanted.** Structured bug reports find root causes in
  batches, not one at a time.
- **Debug the data, not the renderer.** The visualizer is a symptom; the
  spectrum is the disease.
- **Checkpoint before refactor.** The architecture review is the rubric;
  a refactor that doesn't close identified gaps is just a rewrite with new
  unknown gaps.
- **Commit before refactor.** The before-state gives the after-state meaning.
  `git diff v8-baseline..v9-step1` is the teaching material.
- **IP due diligence before publishing.** Renamed from the Cthugha lineage
  before the first public commit — no history to explain.
- **Questions about correctness come before questions about completeness.**
  Build, validate, audit, document, commit.

---

*Update §1 and §5 at the end of every working session. If §1 disagrees with
the repo, the repo wins — then fix this file.*
