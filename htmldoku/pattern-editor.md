# Pattern Editor

The Pattern Editor is Zynthian's built-in step sequencer. It lets you program note patterns and CC automation directly on the device, without an external computer or MIDI file.

---

## Overview

The pattern editor works with the **step sequencer** (`zynseq`). Patterns are attached to chains — each chain can have multiple patterns (phrases). The launcher screen (mixer tab 2) shows all patterns as a grid of pads.

There are two views, switchable via tabs:
- **Notes view** (`pated_notes`) — place and edit notes on a pitch/step grid
- **CC view** (`pated_cc`) — add CC automation events on a CC number/step grid

---

## Opening the Pattern Editor

**From the touch keypad:** Tap **PAD/STEP** (short) to open the Launcher. Tap any clip pad to select it, then long-press the pad → **Edit**, or tap the pencil icon in the toolbar.

**Direct shortcut:** Tap **PAD/STEP** (bold hold, 300ms) to jump directly to the Pattern Editor for the currently selected clip.

**V5 hardware encoder:** With a clip pad selected, bold-press encoder 3.

---

## Notes Grid

```
Octave  Pitch  │ 1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 │
  5      C5    │ [██████  ]           [██  ]               [████  ] │
  5      B4    │          [█  ]                                     │
  5      A4    │ [████████████████████████████████████████████████] │
  5      G4    │                    [██  ]                          │
  5      F4    │                              [██  ]                │
  5      E4    │                                                     │
  5      D4    │                                                     │
  5      C4    │                                                     │
```

- **Rows** — pitch (note), one row per semitone. Rows scroll vertically; octave and note name shown at left.
- **Columns** — steps (time positions), left to right. Scroll horizontally for longer patterns.
- **Colored bars** — notes. Width = duration (how many steps the note lasts). Color may indicate velocity or selection state.

### Note States

| Bar style | State |
|-----------|-------|
| Solid filled bar | Normal note |
| Dotted/outlined bar | Note copied to clipboard (not yet pasted) |
| Bright/different color | Selected note |

---

## Adding and Removing Notes

| Action | Result |
|--------|--------|
| Tap empty cell | Add a note at that pitch and step (default duration 1 step, velocity 100) |
| Tap existing note | Select it (highlights) |
| Tap selected note | Deselect |
| Drag right from a note | Extend the note duration |
| Long-press a note (~300ms) | Open per-note parameter editor |
| Double-tap a note | Delete it |
| Drag on empty area | Box select multiple notes |
| Encoder 4 rotate (note selected) | Adjust selected note's duration |

---

## Per-Note Parameters

Long-press any note to open the per-note parameter editor. All parameters:

| Parameter | Range | Description |
|-----------|-------|-------------|
| **Duration** | 1 – pattern length | Length of the note in steps |
| **Velocity** | 1–127 | Note-on velocity; higher = louder |
| **Offset** | 0–100% | Nudge note start within the step — useful for swing and groove |
| **Stutter Speed** | 1–16 | How many times the note repeats (stutters) within one step duration |
| **Stutter VFX** | FLAT / FADE-IN / FADE-OUT | Velocity envelope for stutter repeats |
| **Stutter Ramp** | NONE / SPEED-UP / SPEED-DOWN | Speed change across stutter repeats |
| **Play Frequency** | NEVER / ALWAYS / PLAY/2–8 / SKIP/2–8 | How often this note fires across pattern passes |
| **Play Chance** | 0–100% | Probability the note plays each time it is reached |
| **Stutter Frequency** | Same options as Play Frequency | How often the stutter fires per play |
| **Stutter Chance** | 0–100% | Probability the stutter fires when note plays |

### Play Frequency Values

| Value | Behavior |
|-------|---------|
| NEVER | Note never plays |
| ALWAYS | Note plays every pass (default) |
| PLAY/2 | Plays on every other pass (1st, 3rd, 5th…) |
| SKIP/2 | Skips every other pass (plays on 2nd, 4th, 6th…) |
| PLAY/3 | Plays once every 3 passes |
| SKIP/3 | Skips once every 3 passes |
| PLAY/4 – PLAY/8 | Plays once every N passes |
| SKIP/4 – SKIP/8 | Skips once every N passes |

These create rhythmic variation — place a hi-hat note at PLAY/2 to create a half-time feel, or a ghost note at PLAY/4 to add occasional embellishments.

### Stutter VFX Options

| Value | Effect |
|-------|--------|
| FLAT | All stutter repeats at same velocity |
| FADE-IN | Velocity ramps up across repeats |
| FADE-OUT | Velocity ramps down (typical decay stutter) |

### Stutter Ramp Options

| Value | Effect |
|-------|--------|
| NONE | All repeats at same speed |
| SPEED-UP | Repeats get faster (accelerando stutter) |
| SPEED-DOWN | Repeats get slower (rallentando stutter) |

Source: [`zyngui/zynthian_gui_pated_notes.py:46`](../zynthian-ui/zyngui/zynthian_gui_pated_notes.py)

---

## Scale Modes

When a scale is active, the note grid shows only the notes in that scale — non-scale notes are hidden. New notes snap to the nearest scale degree.

| Scale | Intervals from root |
|-------|-------------------|
| Major | 0, 2, 4, 5, 7, 9, 11 |
| Minor | 0, 2, 3, 5, 7, 8, 10 |

Activate via the scale selector control in the toolbar. Choose the root note (C–B) then the scale type.

---

## Chord Modes

Instead of placing single notes, chord mode places multiple simultaneous notes when you tap a single cell.

| Chord mode | What gets placed |
|------------|----------------|
| **Single note** | One note (default) |
| **Chord** | The selected chord type at the tapped pitch |
| **Diatonic triads, major key** | 3-note chord built on the tapped scale degree (major key harmony) |
| **Diatonic 7ths, major key** | 4-note chord built on the scale degree (major key) |
| **Diatonic triads, minor key** | 3-note chord (minor key harmony) |
| **Diatonic 7ths, minor key** | 4-note chord (minor key) |

### Available Chord Types

| Category | Chord | Intervals |
|----------|-------|-----------|
| **Triads** | Major | 0, 4, 7 |
| | Minor | 0, 3, 7 |
| | Diminished | 0, 3, 6 |
| | Augmented | 0, 4, 8 |
| **7th chords** | Major 7th | 0, 4, 7, 11 |
| | Minor 7th | 0, 3, 7, 10 |
| | Dominant 7th | 0, 4, 7, 10 |
| | Half-Diminished 7th | 0, 3, 6, 10 |
| | Diminished 7th | 0, 3, 6, 9 |
| | Minor-Major 7th | 0, 3, 7, 11 |
| | Augmented Major 7th | 0, 4, 8, 11 |
| | Augmented 7th | 0, 4, 8, 10 |
| **Extended** | Major 9th | 0, 4, 7, 11, 14 |
| | Dominant 9th | 0, 4, 7, 10, 14 |
| | Minor 9th | 0, 3, 7, 10, 14 |
| | Minor-Major 9th | 0, 3, 7, 11, 14 |
| | Dominant 11th | 0, 4, 7, 10, 14, 17 |
| | Minor 11th | 0, 3, 7, 10, 14, 17 |
| | Dominant 13th | 0, 4, 7, 10, 14, 17, 21 |
| | Minor 13th | 0, 3, 7, 10, 14, 17, 21 |

Tap one cell in chord mode — the full chord appears as a stack of staggered simultaneous notes. All notes in the chord are affected by the same per-note parameters (velocity, offset, etc.).

---

## Transport Controls

The toolbar shows transport buttons:

| Control | Action |
|---------|--------|
| ▶ Play | Start pattern playback from current position |
| ⏸ Pause | Pause at current position |
| ⏹ Stop | Stop and return to step 1 |
| 🔁 Loop | Toggle loop mode (pattern repeats indefinitely) |
| Metronome | Toggle click track during playback |
| Tempo | Current BPM (same as global) — tap to open tempo screen |
| Length | Steps in pattern — tap to change (4–256 steps) |

Playback synchronizes to the global Zynthian clock. If JACK transport is running, patterns follow it.

**Touch keypad transport:**

| Action | Touch keypad |
|--------|-------------|
| Start playback | **PLAY (▶)** in bottom row |
| Stop playback | **STOP (■)** in bottom row |
| Return to Launcher | **BACK/NO** |
| Return to Mixer | **MIX/LEVEL** (short) |

---

## Pattern Length

Configurable from 1 to 256 steps per pattern. Common lengths:

| Steps | Duration at 16th-note resolution |
|-------|----------------------------------|
| 8 | Half bar (4/4) |
| 16 | 1 bar |
| 32 | 2 bars |
| 64 | 4 bars |
| 128 | 8 bars |

Change via the Length control in the toolbar. Notes beyond the new length are hidden but not deleted — extending back recovers them.

---

## CC Automation Editor

The CC tab (`pated_cc`) shows the same step grid but rows represent MIDI CC numbers instead of pitches.

| CC# | Common function |
|-----|----------------|
| 1 | Modulation wheel |
| 7 | Channel volume |
| 10 | Pan |
| 11 | Expression |
| 64 | Sustain pedal |
| 71 | Resonance (filter) |
| 74 | Brightness (filter cutoff) |
| 91 | Reverb send |
| 93 | Chorus send |

### CC Editor Interactions

| Action | Result |
|--------|--------|
| Tap empty step on a CC row | Add a CC event at default value (64) |
| Drag up/down on a CC event | Adjust CC value (0–127) |
| Long-press a CC event | Open value editor for exact value |
| Double-tap a CC event | Delete |
| Drag across multiple steps | Lasso-select for batch edit |

---

## Copy and Paste

**Select notes:** touch and drag to draw a selection box. All notes within are selected (bright highlight).

**Copy selected notes:** copy button in toolbar.

**Paste:** tap target position in the grid → paste button. Notes paste relative to the top-left of the selection box.

**Copy between channels:** switch to another chain's pattern (back to launcher → select different clip → open editor) and paste there.

---

## Example: 8-Step Bass Line on setBfree Organ

1. Add Instrument chain → setBfree. Assign to MIDI channel 2.
2. Tap a clip pad for chain 2 → open pattern editor.
3. Set pattern length to 8 steps.
4. Place notes on A minor pentatonic: A2 (step 1 duration 2), G2 (step 3 duration 1), F2 (step 4), E2 (step 5 duration 2), G2 (step 7), A2 (step 8).
5. Set velocity: step 1 = 110 (accent), others = 80.
6. Set stutter on step 5: Stutter Speed = 2, VFX = FADE-OUT — creates a ratchet effect.
7. Press Play — the bass line loops.

---

## Example: Drum Pattern — FluidSynth GM Drums

GM General MIDI drum map note assignments (channel 10):

| MIDI note | Note name | Drum sound |
|-----------|-----------|-----------|
| 35 | B1 | Bass drum 2 |
| 36 | C2 | Bass drum 1 |
| 38 | D2 | Acoustic snare |
| 40 | E2 | Electric snare |
| 42 | F#2 | Closed hi-hat |
| 44 | G#2 | Pedal hi-hat |
| 46 | A#2 | Open hi-hat |
| 49 | C#3 | Crash cymbal 1 |
| 51 | D#3 | Ride cymbal 1 |
| 48 | C3 | Hi tom |
| 45 | A2 | Low tom |

Classic 4/4 pattern (16 steps):

| Step | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 |
|------|---|---|---|---|---|---|---|---|---|----|----|----|----|----|----|-----|
| Bass | ● | | | | ● | | | | ● | | | | ● | | | |
| Snare | | | | | ● | | | | | | | | ● | | | |
| HH | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |

1. Add Instrument chain → FluidSynth → "General MIDI" bank → assign to channel 10.
2. Open pattern editor. Set to 16 steps.
3. Place C2 (bass drum) at steps 1, 5, 9, 13.
4. Place D2 (snare) at steps 5, 13.
5. Place F#2 (hi-hat closed) every step.
6. Set hi-hat velocity to 60 (quieter than kick and snare).
7. Add open hi-hat (A#2) at step 7 with velocity 80 — gives a "2 and" feel.
8. Press Play.

---

## What's Next

- [Synth Engines](synth-engines.html) — engines to pair with the sequencer
- [Chains & Routing](chain-management.html) — set up the instrument chain
- [Snapshots](snapshots.html) — save a complete sequenced setup
- [UI Navigation](ui-navigation.html) — launcher screen and clip pads

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_pated_notes.py`, `zyngui/zynthian_gui_pated_cc.py`.*
