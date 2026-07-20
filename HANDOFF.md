# HANDOFF — Signal2Noise AudioViz

> **First action for any fresh session (human or AI): read this file top to bottom.**
> This is a static GitHub Pages repo — no dev server, so there is no `GET /HANDOFF.md`
> or `GET /status`; this file *is* the status endpoint.

---

## 1. Current state

- **Current version:** `v9/signal2noise_v9.html` — v9 step 5 (floating tools)
  - step 1 renderer extraction: ✅ committed, checkpoint passed
  - step 2 spiral view: ✅ checkpoint passed (feedback engine, bass→rotation, treble/mic→zoom)
  - step 3 tool tray: gate passed, ⬜ browser checkpoint pending
  - step 4 channel strip + per-channel intensity: ✅ checkpoint passed
  - step 4.1 fire ambient-gating fix: gate passed, ⬜ browser checkpoint pending.
    v8 latent behavior: ambient spectrum→heat injected unconditionally, so the
    fire pill looked permanently on. Now ambient heat only lands inside
    fire-enabled channels' x-regions (chanVfx added to shared read-only inputs);
    mic heat requires ≥1 fire-enabled channel. Deliberate v8-behavior change.
  - step 5 floating tools: superseded by step 6 same-session (user wanted
    docked, not floating). Kept: mesh-gradient grip, opacity slider,
    FH 220→440, ✕ / `tools` button / `h`.
  - step 6 docked tray: gate passed, ⬜ browser checkpoint pending.
    Bottom-docked drawer, full-bleed app (viz fills viewport behind it).
    Grip drag = ns-resize (clamped grip-height…92vh); drag-collapse preserves
    the restore height (bug caught by gate: trayH was being clobbered);
    dblclick grip = quick collapse/expand; `auto` button = collapse on
    pointerleave, expand on pointerenter. Canvas now CSS-stretched to fill
    (non-uniform scale — Cthugha-appropriate; revisit if it bothers).
    Perf watch stands: fire automaton at 2× cells.
- **Frozen baseline:** `v8/signal2noise_v8.html` — do not modify; it exists so `git diff` tells the refactor story
- **Behavior contract for step 1:** visually and audibly identical to v8
- **Validation done:** headless Node.js simulation (stubbed DOM/Canvas/WebAudio) — load+init, dispose→init lifecycle, dispatch→bus→onTrigger→tick full frame, all 6 VFX paths incl. balloon-pop over 120 frames, and exactly **1** trigger listener on the bus after repeated view switches (no leak)
- **Checkpoint status:** step 3 (tray) pending; steps 1–2 passed in browser

### Known issues

- **Mic "arm mic" permission error** (observed after v9 step 1 checkpoint). Sampler
  code is a verbatim move from v8, so this is almost certainly environmental —
  browser permission previously denied for the origin, or OS mic privacy settings.
  Not a current priority. Revisit before any sampler work.

## 2. File map

```
README.md                        project front door; history table; roadmaps
HANDOFF.md                       this file — state, contract, workflow, backlog
signal2noise-audioviz.html       the app — current working file (single-file, zero-dependency)
v8/signal2noise_v8.html          frozen pre-refactor baseline
architecture/architecture_review.html   v8 rubric: GoF / SOLID / 22 missing tests
```
Branch: `main` (master deleted). Live demo is served by GitHub Pages from main
at /signal2noise-audioviz.html — there is no index.html, so the bare Pages URL
404s by design; README links the full path.

Everything ships as single-file HTML. No build step. proto2prod rules apply:
validate before optimize, surgical edits, visible errors over graceful degradation.

## 3. Architecture contract (v9)

### Renderer contract — every view implements exactly this

```
init(env)       env = {ctx, vx, FW, FH}. Build ALL internal state fresh.
onTrigger(e)    e = {ci, cx, xFrac, width, intensity, vol, col, vfxSet, viz, t}
                One sequencer hit, timing already compensated. React or ignore.
                e.viz = per-channel effective intensity: vizIntensity if the
                channel participates (chanIntensity[ci]), else 1.0. Use e.viz
                for trigger-born visuals; global vizIntensity stays for
                ambient/whole-frame effects (Alpha's spectrum heat, Spiral's
                spectrum arm, creature render scale).
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
6. When a version is complete: update the README history table and this file's
   §1/§5, then tag and release: `git tag <tag> && git push --tags`, GitHub →
   Releases → attach `signal2noise-audioviz.html` → publish. README links
   /releases/latest, so no README edit is needed per release.

## 5. Backlog (ordered)

1. ✅ Checkpoint v9 step 1 — passed (boom matched side by side)
2. ✅ Commit step 1; `index.html` redirect → v9 — done
3. ✅ **Spiral view** — shipped + checkpoint passed
4. ✅ Tool tray (step 3) — superseded by step 5's floating tray
5. ✅ **Channel strip** (step 4) — checkpoint passed. Click channel NAMES to
   select; tri-state pills + volume + ⚡int apply to selection; excluded
   channels dim and pin at e.viz=1.0.
5b. ⬜ **Step 4.1 + 5 checkpoint** — fire pill actually kills fire per-channel;
   tray drags/opacity/toggles; viz at 2× height holds framerate.
6. ⬜ Sand view — particle deposition, per-channel pour points, heightmap + angle of repose
7. ⬜ Organic view — metaballs via radial gradients + threshold pass
8. ⬜ Galactic view — softened n-body, triggers add mass/velocity
9. ⬜ Synth-side Strategy — `synth()` switch → voice map (arch review finding)
10. ⬜ FM voices + synth panel — 2-op FM (mod → gain(index) → carrier.freq),
    dials: ratio / index / decay
11. ⬜ Write the 22 missing tests (arch review) on the Node-simulation scaffolding
12. ⬜ Command pattern — preset actions + undo stack
13. ⬜ State machines — sequencer + sampler

### 5a. Channel strip — decision record (resolved)

Selection is the editing TOOL (transient); it sets two INDEPENDENT per-channel
flags, which may overlap freely:
- chanVfx[ci] — which elements a hit adds to the visualization (Alpha pills)
- chanIntensity[ci] — whether the channel rides the global intensity slider
Exception gets the marker: intensity-excluded names render dimmed; the
all-on default stays visually quiet. No arm-and-fire mechanics (proto2prod).

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
