# Signal2Noise AudioViz

A browser-based audio visualizer + drum sequencer, built as a prototype-to-production teaching project.

Originally inspired by Cthugha (1993), a classic DOS fire-and-waveform visualizer by Kevin "Zaph" Burfitt. Cthugha's engine was feedback-based — pixel translation tables and palette rotation — a lineage that continues directly into the upcoming Spiral view.

## Live demo

**[▶ Launch Signal2Noise AudioViz](https://meatpopsci1972.github.io/Signal2Noise_AudioViz/)**

Hosted via GitHub Pages. Click play, then try a preset — no install, no build step.

---

## What it does

- **16-channel drum sequencer** — synthesized drums via Web Audio API, swing, 16/32 step modes
- **View system (new in v9)** — tab bar of visualizer views built on a Renderer strategy contract
- **Alpha view** — the original fire engine: cellular automaton pixels, each channel burns its own frequency column
- **6 VFX types per channel** — fire, strobe rings, balloons, sparkles, fireworks (googly-eyed), animorphic creatures
- **Animorphic creatures** — persistent bezier blobs that walk a mountain silhouette, pulse on beat, blink, open their mouths
- **Mic sampler + preset builder** — record audio, onset detection + spectral centroid classifier auto-builds a drum pattern
- **5 built-in presets** — boom bap, house, jungle, hip-hop, techno + random + clear
- **Intensity slider** — uniform visual amplification across all layers

---

## Repository structure

```
Signal2Noise_AudioViz/
├── README.md                          ← this file
├── HANDOFF.md                         ← session continuity: state, contract, backlog
├── index.html                         ← redirect to current version
├── v8/
│   └── signal2noise_v8.html           ← frozen baseline (pre-refactor)
├── v9/
│   └── signal2noise_v9.html           ← current: renderer extraction + view tabs
└── architecture/
    └── architecture_review.html       ← GoF / SOLID / test gap analysis (v8 rubric)
```

---

## Prototype history

| Version | What changed |
|---------|-------------|
| v1 | Basic Cthugha-style fire visualizer, keyboard input |
| v2 | Keyboard drum layout, frequency-mapped fire columns |
| v3 | Per-note VFX: rings, vignettes, beam columns, drum-typed visuals |
| v4 | 16-channel step sequencer, synthesized drums, 5 presets |
| v5 | Mic sampler + onset detection preset builder |
| v6 | **Bug fix:** preset switching — 5 scheduler/state bugs found and fixed via Node.js simulation |
| v7 | **Bug fix:** fire visualization — analyser stride bug, per-channel direct heat injection, spectral centroid classifier |
| v8 | 6 VFX types per channel (pills UI), animorphic creatures, mountain silhouette, fireworks with googly eyes. **Frozen as pre-refactor baseline.** |
| v9 step 1 | **Refactor:** Renderer contract + EventBus. `trigViz()` deleted; all visual state moved into `createAlphaRenderer()`; globals 21 → 12; view tab bar added. Validated headless via Node.js lifecycle simulation. Behavior identical to v8 by contract. |

---

## Architecture

The v8 baseline analysis lives in [`architecture/architecture_review.html`](architecture/architecture_review.html) — 10 subsystems, 21 globals, 5 SOLID violations, 8 prioritized GoF patterns, 22 missing tests. It is the rubric v9 is measured against.

### v9 refactor progress

| Finding | Status |
|---------|--------|
| Strategy — renderer side | ✅ step 1: `createAlphaRenderer()` behind init/onTrigger/tick/dispose |
| Observer / EventBus | ✅ step 1: scheduler emits `trigger`; `trigViz()` deleted |
| Strategy — synth side | ⬜ planned (FM synth track) |
| Template Method — unified VFX lifecycle | ◐ partially absorbed by the renderer contract; revisit inside Alpha |
| Command — preset actions + undo | ⬜ planned |
| Facade — injectable interfaces for tests | ◐ frame-object injection exists; formalize with the test pass |
| 22 missing tests | ⬜ Node simulation scaffolding exists; tests not yet written |
| State machines — sequencer + sampler | ⬜ planned |

### View roadmap

| View | Status | Core idea |
|------|--------|-----------|
| alpha | ✅ shipped (v9 step 1) | Cthugha-lineage fire + VFX + creatures (the v8 look) |
| spiral | next | Feedback buffer with pixel ghosting — no clear, translucent fade + rotate |
| sand | planned | Particle deposition; channels pour colored grains, the pile is the song's history |
| organic | planned | Metaballs; channel blobs grow on triggers and absorb each other |
| galactic | planned | Softened n-body; mass centers per channel group, triggers add mass and velocity |

All views consume the same inputs — per-channel trigger events + a per-frame spectrum — which is exactly what the Renderer contract formalizes.

### Sound roadmap

FM synthesis controls (separate synth panel): two-operator FM voices — modulator → gain (index) → carrier frequency — with ratio / index / decay dials. Requires the synth-side Strategy refactor first.

---

## Prototype-to-production teaching notes

This project deliberately preserves its full iterative history — bugs found, tests written after the fact, architectural debt identified before refactoring. The distilled discipline, and the prompts that drove each phase, now live in [`HANDOFF.md`](HANDOFF.md) alongside the operational state. The short version:

1. **Rapid prototyping is valid** — the fire engine, sequencer, and VFX system were all built fast and flat
2. **Validation before production** — real bugs caught by simulating logic in Node.js *before* shipping (5 in v6, 3 pre-baseline, lifecycle seams in v9)
3. **Architecture review as a checkpoint** — GoF gaps and SOLID violations identified before committing to a refactor
4. **Commit before refactor** — v8 is the frozen before-state; the v8→v9 diff *is* the teaching material:

```
git diff v8-baseline..v9-step1
```

---

## Running it

Open `v9/signal2noise_v9.html` directly in any modern browser. No build step, no dependencies. (`v8/signal2noise_v8.html` remains as the frozen baseline for diffing.)

Click the visualizer first to unlock the AudioContext, then hit **play** or click a preset.

For mic input / sampler: click **arm mic** and grant browser microphone permission.

Clicking the active view tab re-runs the renderer's dispose→init cycle — a built-in lifecycle self-test.

---

## Tech stack

- Vanilla JS (no framework)
- Web Audio API — synthesis, scheduling, analysis
- Canvas 2D API — fire pixel buffer + VFX overlay
- MediaRecorder API — mic capture
- No build tools, no bundler, no dependencies
