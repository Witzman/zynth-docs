# Design: SMC-PAD Drum Computer

**Date:** 2026-05-29  
**File target:** `htmldoku/project-drum-computer.md`  
**Difficulty:** Intermediate  
**Access:** SSH · Webconf · VNC

---

## Goal

Use all 16 SMC-PAD pads as a drum computer: pads 1–12 trigger GM drum sounds live, pads 13–16 launch/stop looped beat patterns. Built on a single FluidSynth chain with a GM drumkit preset.

---

## Parts

### Part 1 — Live drum kit
- Create a FluidSynth chain set to the SMC-PAD's MIDI channel (channel 7 in Preset 1)
- Load a GM drumkit preset
- Verify: pressing any pad produces a drum sound

### Part 2 — Program a beat
- Open the pattern editor for the drum chain
- Build a simple 4/4 beat via grid-based step entry (no MIDI recording needed)
- Play it back via the sequencer
- Verify: pattern loops correctly

### Part 3 — Launch patterns from pads
- Build a 1 chain × 4 phrases launcher layout (4 pattern slots)
- Map top-row pads 13–16 (notes 48–51) to TOGGLE_SEQ 0,0–0,3 via master key actions
- Pads 1–12 remain live drum hits
- Verify: top-row pads launch/stop patterns while remaining pads play live drums simultaneously

### Going Further
- Custom samples via SFZ (LinuxSampler) — replace GM soundfont with user .wav files
- SMC-PAD bank 2 for scene switching (16 more pattern slots)
- Save full setup as named snapshot for instant recall

---

## MIDI Routing

**Live play:**  
SMC-PAD Preset 1 → notes 36–51 on channel 7 → FluidSynth chain (listening on channel 7) → GM drum sounds. Notes 36–51 align with standard GM drum map (kick=36, snare=38, hats=42/44/46, cymbals=49/51).

**Launcher control:**  
Master MIDI channel set to 7 (matches SMC-PAD). Pads 13–16 (notes 48–51) intercepted by master key actions → TOGGLE_SEQ 0,0 through 0,3. Master key action notes are consumed by the UI layer and not passed to the engine.

**Open verification item:**  
Confirm master key action notes are consumed (not passed to FluidSynth chain). If they pass through, top-row pads will play drum sounds AND toggle patterns simultaneously.

---

## Sequencer Integration

- Single drum chain with multiple phrases (patterns)
- Patterns built in the grid-based pattern editor via VNC/touchscreen
- Launcher view: 1 chain × 4 rows = 4 slots
- Live MIDI input and looping pattern playback coexist on the same chain
- Verify: no timing conflicts or double-triggering between live input and pattern loop

---

## Snapshot Strategy

- Tutorial is a standalone setup — does not conflict with the SMC-PAD Launcher tutorial snapshot
- Final step of Part 3: save named snapshot via webconf → Library → Snapshots
- Recall restores all chains, patterns, and MIDI mappings in one step

---

## Key Constraints

- All steps achievable via SSH, webconf, or VNC — no physical encoders or buttons assumed
- SMC-PAD accessed via USB (class-compliant, no driver)
- Preset 1 note/channel values are verified; other presets need re-verification via `amidi -d`
- Tutorial references SMC-PAD Launcher tutorial for pad note/channel discovery (Part 1 of that tutorial)
