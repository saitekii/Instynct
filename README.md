# Instynct — Musical Fluency Trainer

> **Hear it. Feel it. Play it.**

A browser-based ear training tool for musicians. Instynct generates short melodic phrases on demand. You listen, you reproduce them on your instrument by ear — no notation required. Repeated exposure to varied, structured phrases builds auditory vocabulary, improvisational instinct, and expressive fluency.

---

## What it does

### Single phrase mode
Press **Generate & Play** to hear a new phrase. Replay it as many times as you need, slow it down to 60%, then try to reproduce it on your instrument before revealing the notes.

### Flow mode
Press **Flow Mode** to start an uninterrupted practice loop:

```
▶ phrase plays  →  equal silence (your turn)  →  new phrase  →  repeat
```

The cycle continues until you stop it. Each phrase is freshly generated. The progress bar shows exactly where you are in the current cycle.

---

## Controls

| Control | What it does |
|---|---|
| **Generate & Play** | Generate and immediately play a new phrase |
| **Replay** | Replay the current phrase at full speed |
| **60% Speed** | Replay at 60% tempo for closer listening |
| **Reveal Notes** | Show the note names and durations (hidden by default) |
| **≈ Drone** | Toggle a sustained root-note drone in the selected key |
| **♩ Metronome** | Toggle a click track (active during Flow Mode) |
| **⬡ Drum Beat** | Toggle a drum pattern (active during Flow Mode) |
| **◫ Bars & Beat** | Show bar lines and a moving playhead on the note display |
| **⟳ Flow Mode** | Start / stop the continuous listen–respond loop |

---

## Settings

| Setting | Options |
|---|---|
| **Key** | All 12 chromatic keys |
| **Scale** | Major, Natural Minor, Dorian, Mixolydian, Phrygian, Penta Major, Penta Minor, Blues, Harmonic Minor, Whole Tone |
| **Tempo** | 40 – 240 BPM |
| **Beats** | 4 / 8 / 12 / 16 beats per phrase |
| **Feel** | Straight, Swing, Sparse, Busy |
| **Beat** | Rock, Funk, Jazz, Bossa Nova, Half-time |
| **Sound** | Flute, Pure Tone, Reed, Bell |

---

## How the melody generator works

Phrases are built in four stages:

1. **Contour** — a shape (`arch`, `valley`, `ascending`, `wave`, etc.) is chosen and mapped to a normalised height curve across all notes, with small random jitter added to interior points.

2. **Rhythm** — durations are assembled from pre-defined 2-beat cells (e.g. `[1, 0.5, 0.5]`, `[1.5, 0.5]`) so every note always lands on a musically valid grid position.

3. **Candidate scoring** — for each note position, 5 candidate pitches near the contour target are scored on four axes:
   - **Voice leading** — steps and thirds preferred over leaps
   - **Chord tone quality** — root, 3rd, and 5th score higher
   - **Beat enforcement** — non-chord tones are penalised on beats 1 and 3
   - **Contour fidelity** — small cost for drifting from the register target

   Selection is weighted-random (squared weights), so the best note wins most of the time while preserving variety.

4. **Post-processing** — large leaps are smoothed, and the final note is nudged to the nearest stable scale degree (root, 3rd, or 5th).

---

## Audio architecture

All sound sources route through a shared master bus:

```
melody synth  ─┐
drone oscs    ─┼─▶  Gain (−8 dB)  ─▶  Limiter (−2 dBFS)  ─▶  output
metronome     ─┤
drum voices   ─┘
```

The gain stage ensures the combined signal arrives at the limiter at a safe level. Drum voices use `MembraneSynth` (kick), `NoiseSynth` with pink noise (snare, ride), and `NoiseSynth` with white noise (hi-hat) — avoiding the CPU cost of `MetalSynth`.

The metronome and all drum patterns share a single `Tone.Transport` instance, keeping them locked in sync.

---

## Running it

No build step, no dependencies to install. Just open `index.html` in a browser.

Tone.js is loaded from CDN — an internet connection is required on first load.

---

## Stack

- Vanilla HTML / CSS / JavaScript
- [Tone.js v14](https://tonejs.github.io/) for all audio synthesis and scheduling
