# Signal2Noise AudioViz

A browser-based audio visualizer + drum sequencer, built as a prototype-to-production teaching project.

Originally inspired by Cthugha (1993), a classic DOS fire-and-waveform visualizer by Kevin "Zaph" Burfitt.
## Live demo

**[▶ Launch Signal2Noise AudioViz v8](https://meatpopsci1972.github.io/Signal2Noise_AudioViz/)**

Hosted via GitHub Pages. Click play, then try a preset — no install, no build step.


---

## What it does

- **16-channel drum sequencer** — synthesized drums via Web Audio API, swing, 16/32 step modes
- **Fire visualizer** — cellular automaton pixel engine, each channel burns its own frequency column
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
├── v8/
│   └── signal2noise_v8.html           ← current working prototype (v8)
└── architecture/
    └── architecture_review.html       ← GoF / SOLID / test gap analysis
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
| v8 | 6 VFX types per channel (pills UI), animorphic creatures, mountain silhouette, fireworks with googly eyes |

---

## Architecture analysis (pre-refactor baseline)

Full analysis is in [`architecture/architecture_review.html`](architecture/architecture_review.html).

### Summary

**10 subsystems, 21 global variables, god-object script architecture.**

The code works well as a prototype but has clear production gaps:

| Issue | Detail |
|-------|--------|
| Single Responsibility | `trigViz()` has 7 responsibilities |
| Open/Closed | New VFX type = 4 files touched |
| Dependency Inversion | Scheduler calls all subsystems by name |
| No unit tests | 22 identified missing tests |
| Mixed object lifecycles | `vfxObjs`, `balloons`, `creatures` in 3 separate loops |

### GoF pattern opportunities (prioritized)

| # | Pattern | Why |
|---|---------|-----|
| 1 | **Strategy** | Kills `synth()` switch + `vfxTick()` if/else. New type = 1 file. |
| 2 | **Observer / EventBus** | Deletes `trigViz()`. Scheduler decoupled from visuals. |
| 3 | **Template Method** | Unifies 3 VFX loops. One lifecycle contract. |
| 4 | **Command** | Preset actions become typed objects. Undo/redo free. |
| 5 | **Facade** | Enables unit testing each subsystem in isolation. |
| 6 | **State Machine** | Explicit sequencer + sampler state transitions. |
| 7 | **Object Pool** | GC pressure at high intensity. Defer until felt. |
| 8 | **Builder** | Preset construction with validation. Nice-to-have. |

---

## Prototype-to-production teaching notes

This project deliberately preserves the full iterative history — bugs found, tests written after the fact, architectural debt identified before refactoring. The intent is to show:

1. **Rapid prototyping is valid** — the fire engine, sequencer, and VFX system were all built fast and flat
2. **Validation before production** — 5 real bugs caught by simulating the state machine in Node.js *before* shipping
3. **Architecture review as a checkpoint** — identifying GoF gaps and SOLID violations before committing to a refactor
4. **Commit before refactor** — this snapshot is the baseline; v9 will apply the patterns and the diff tells the story

---

## Running it

Open `v8/signal2noise_v8.html` directly in any modern browser. No build step, no dependencies.

Click the visualizer first to unlock the AudioContext, then hit **play** or click a preset.

For mic input / sampler: click **arm mic** and grant browser microphone permission.

---

## v9 refactor plan (upcoming)

- [ ] Strategy pattern — synth map + VFX update map
- [ ] EventBus — decouple scheduler from all visual subsystems
- [ ] Template Method — unified VFX lifecycle loop
- [ ] Command pattern — preset actions + undo stack
- [ ] Facade — injectable interfaces for unit testing
- [ ] Write the 22 identified missing tests
- [ ] State machines for sequencer + sampler

---

## Tech stack

- Vanilla JS (no framework)
- Web Audio API — synthesis, scheduling, analysis
- Canvas 2D API — fire pixel buffer + VFX overlay
- MediaRecorder API — mic capture
- No build tools, no bundler, no dependencies
