# Pattern Editor

The Pattern Editor is Zynthian's built-in step sequencer. It lets you program note patterns and CC automation directly on the device, without an external computer.

---

## Opening the Pattern Editor

**From the launcher screen:** each clip pad in the launcher represents one pattern. Tap a pad to select it, then open the pattern editor via the edit button (or long-press the pad).

**From Admin:** Admin → sequence / pattern editor (when available in the menu).

The pattern editor opens in the **notes view** (`pated_notes`). Switch to CC view via the tab at the top.

---

## Notes Grid

```
     Step: 1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16
C5       [ ██          ]  [ █  ]                 [ ██████ ]
B4              [ █  ]
A4       [ ██████████████████████████████████████████████ ]
G4                        [ ██  ]
F4                                    [ ██  ]
```

- **Rows** = pitch (note), highest at the top
- **Columns** = steps (time positions)
- **Colored bars** = notes, width = duration

The grid scrolls both horizontally (more steps) and vertically (more notes).

### Adding and Removing Notes

| Action | Effect |
|--------|--------|
| Tap empty cell | Add a note at that pitch and step |
| Tap existing note | Select it |
| Drag right on a note | Extend duration |
| Long-press a note | Open per-note parameter editor |
| Double-tap a note | Delete it |

---

## Per-Note Parameters

Long-press any note to open the parameter editor. Parameters:

| Parameter | Description |
|-----------|-------------|
| **Duration** | Length of the note in steps (1 = one step) |
| **Velocity** | Note-on velocity (1–127) |
| **Offset** | Nudge the note start within the step (for swing/groove) |
| **Stutter Speed** | How fast the note stutters (repeats within the step) |
| **Stutter VFX** | Velocity envelope for stutter: FLAT, FADE-IN, FADE-OUT |
| **Stutter Ramp** | Speed ramp for stutter: NONE, SPEED-UP, SPEED-DOWN |
| **Play Frequency** | How often this note plays: ALWAYS, PLAY/2, SKIP/2, PLAY/3… |
| **Play Chance** | Probability this note plays each time it is reached |
| **Stutter Frequency** | How often the stutter fires: same options as Play Frequency |
| **Stutter Chance** | Probability the stutter fires |

**Play Frequency** and **Play Chance** create rhythmic variation — a note set to PLAY/2 fires only on alternate passes through the pattern.

Source: [`zyngui/zynthian_gui_pated_notes.py:46`](../zynthian-ui/zyngui/zynthian_gui_pated_notes.py)

---

## Scale and Chord Modes

The pattern editor can restrict note entry to a musical scale, making it easier to compose in key.

**Scale modes:**

| Scale | Intervals |
|-------|-----------|
| Major | C D E F G A B |
| Minor | C D Eb F G Ab Bb |

When a scale is active, only scale notes are shown as rows, and new notes snap to the nearest scale degree.

**Chord modes:** instead of placing single notes, the editor places full chords:

| Mode | What gets placed |
|------|-----------------|
| Single note | One note |
| Chord | Selected chord type (see below) |
| Diatonic triads, major key | Triad built on the scale degree you tap |
| Diatonic 7ths, major key | 7th chord on the scale degree |
| Diatonic triads, minor key | Triad in minor key |
| Diatonic 7ths, minor key | 7th chord in minor key |

Available chord types include: Major, Minor, Diminished, Augmented, Major 7th, Minor 7th, Dominant 7th, Half-Diminished 7th, and many extended chords (9th, 11th, 13th). Tap one cell — the full chord appears as a stack of simultaneous notes.

---

## Transport Controls

The bottom of the pattern editor shows transport buttons:

| Button | Action |
|--------|--------|
| Play/Pause | Start or pause pattern playback |
| Stop | Stop and return to step 1 |
| Loop | Toggle loop mode (pattern repeats) |
| Metronome | Toggle click track |
| Tempo | BPM (same as global tempo) |

Pattern playback is synchronized to the global clock. If other chains or sequences are running, the pattern editor follows the same tempo.

---

## CC Automation Editor

Switch to the CC tab at the top of the pattern editor (`pated_cc`).

The CC editor shows the same step grid but rows represent CC numbers instead of pitches. Place a CC event at any step with any value — useful for automating filter sweeps, volume curves, or any parameter bound to a CC.

| Action | Effect |
|--------|--------|
| Tap step on a CC row | Add a CC event |
| Drag up/down on a CC event | Adjust the CC value |
| Long-press a CC event | Set exact value |
| Double-tap | Delete the CC event |

---

## Copy / Paste Patterns

Select a range of notes (touch and drag), then use the copy/paste buttons in the toolbar. Paste into the same pattern at a different step, or switch to another channel and paste there.

---

## Pattern Length

The number of steps per pattern is configurable per pattern. Common lengths:

- 16 steps = 1 bar (at 4/4, 16th-note resolution)
- 32 steps = 2 bars
- 64 steps = 4 bars

Change the pattern length via the length control in the toolbar (encoder 4 or touch).

---

## Example: 8-Step Bass Line on setBfree Organ

1. Add an Instrument chain → setBfree.
2. Open pattern editor for the setBfree clip.
3. Set pattern length to 8 steps.
4. Place notes on a minor pentatonic scale: A2 at steps 1, 2, 3; G2 at step 5; F2 at step 7.
5. Set velocities: step 1 = 100, others = 80 for accent on beat 1.
6. Press Play — the bass line loops over setBfree with the organ sound.

---

## Example: Drum Pattern — FluidSynth GM Drums on Channel 10

GM drum map uses fixed notes for each drum:

| Note | MIDI# | Drum |
|------|-------|------|
| C2 | 36 | Bass drum |
| D2 | 38 | Snare |
| F#2 | 42 | Hi-hat closed |
| A#2 | 46 | Hi-hat open |
| C3 | 48 | Tom hi |

1. Add Instrument chain → FluidSynth → General MIDI soundfont → assign to MIDI channel 10.
2. Open pattern editor, set to 16 steps.
3. Place bass drum (C2) at steps 1, 5, 9, 13.
4. Place snare (D2) at steps 5, 13.
5. Place closed hi-hat (F#2) every 2 steps.
6. Press Play — classic four-on-the-floor pattern.

---

## What's Next

- [Synth Engines](synth-engines.html) — engines to pair with the sequencer
- [Chains & Routing](chain-management.html) — set up the instrument chain
- [Snapshots](snapshots.html) — save a complete sequenced setup

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_pated_notes.py`, `zyngui/zynthian_gui_pated_cc.py`.*
