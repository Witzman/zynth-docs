# MIDI Recorder

The MIDI Recorder captures all MIDI activity to a Standard MIDI File (SMF), and plays back `.mid` files through loaded chains. Use it to sketch ideas, record live performances, or replay sequences from an external DAW or notation software.

---

## Accessing the MIDI Recorder

| Method | How |
|--------|-----|
| Main Menu | SW1 short → Main Menu → **MIDI Recorder** (piano-roll icon) |
| CUIA | `SCREEN_MIDI_RECORDER` action (hardware button or webconf mapping) |

---

## Screen Layout

The screen uses the selector-info layout: file list on the left, info panel on the right. A secondary controller (encoder 2) appears at the bottom right — its function changes depending on playback state.

```
┌─────────────────────────────────────────┐
│ ⬤ Start MIDI Recording                  │  ← always first entry
│                                         │
│ SD> Captured MIDI Tracks                │  ← section header
│   2024-11-01_143022_MySong (2:34)       │
│   2024-10-30_095501 (0:47)              │
│                                         │
│ SD> User MIDI Tracks                    │
│   Intro_A (1:12)                        │
│                                         │
│ USB> MyDrive MIDI Tracks                │
│   drum_loop_120bpm (0:08)               │
├─────────────────────────────────────────┤
│ [Loop: off]                             │  ← encoder 2 (idle) or BPM (playing)
└─────────────────────────────────────────┘
```

---

## File Sources

Files are listed sorted by modification time (newest first) within each section.

| Section header | Directory | Contents |
|---------------|-----------|----------|
| **SD> Captured MIDI Tracks** | `$ZYNTHIAN_MY_DATA_DIR/capture/` | Auto-saved recordings |
| **SD> User MIDI Tracks** | `$ZYNTHIAN_MY_DATA_DIR/files/Midi/` | Manually placed MIDI files |
| **USB> [device] MIDI Tracks** | `/media/root/[device]/` | Files on attached USB storage |

Files are displayed as `name (M:SS)` where the duration is read from the SMF file's actual content.

---

## Recording

### Starting a Recording

Tap **⬤ Start MIDI Recording** at the top of the list. The entry changes to **■ Stop MIDI Recording** to confirm recording is active.

All MIDI input arriving at ZynMidiRouter is captured — every channel, every port. The recording accumulates in memory while active.

### Stopping and Saving

Tap **■ Stop MIDI Recording**. The file is written to disk immediately.

**Filename format:**

```
YYYY-MM-DD_HHMMSS.mid
YYYY-MM-DD_HHMMSS_SnapshotName.mid    ← if a snapshot is loaded
```

Example: `2024-11-01_143022_Blue_Sky.mid`

**Save location priority:**
1. First writable USB storage device found (if any attached)
2. Internal SD card: `$ZYNTHIAN_MY_DATA_DIR/capture/`

The new file appears at the top of "SD> Captured MIDI Tracks" (or the USB section) immediately after save.

### Recording Indicator

While recording, the screen header entry shows ■. On V5 hardware, the REC LED turns red.

---

## Playback

### Playing a File

Tap any file entry (short press). Playback starts immediately:

1. Zynthian sets its tempo to match the BPM stored in the SMF file.
2. JACK transport starts.
3. All MIDI channels in the file route through ZynMidiRouter — chains on matching MIDI channels respond.

The playing file is prefixed with **▶** in the list.

Tap the same file again (or tap **Stop** in CUIA) to stop playback.

### Loop Mode

The **Loop** controller (encoder 2, bottom right) toggles loop mode:

| Loop state | Behavior |
|------------|---------|
| **off** | File plays once then stops |
| **on** | File restarts automatically on completion |

Loop state persists across sessions. Stored as env var: `ZYNTHIAN_MIDI_PLAY_LOOP` (0/1).

### Tempo Control During Playback

When a file is playing, encoder 2 changes function from Loop to **BPM control** — rotate to nudge playback tempo up or down in real time. This adjusts the global Zynthian tempo, which also affects the step sequencer if it is running.

### V5 LED States During Playback

| LED | Color | Meaning |
|-----|-------|---------|
| REC | Red | Recording active |
| REC | Alt | Not recording |
| PLAY | Green | Playback active |
| PLAY | Alt | Not playing |
| STOP | Alt | Always (tap to stop via CUIA) |

---

## File Management

**Bold-press** any file entry to open its options:

| Option | Effect |
|--------|--------|
| **Rename** | Opens on-screen keyboard → renames the `.mid` file on disk |
| **Delete** | Confirm dialog → permanently removes the file |

Rename uses the current filename (without `.mid`) as the starting text. The new name replaces the file immediately — no restart needed.

---

## Interactions Summary

| Action | Result |
|--------|--------|
| Tap recording toggle | Start or stop MIDI capture |
| Tap file (short) | Toggle playback of that file |
| Bold-press file | Rename / Delete options |
| Encoder 2 rotate (idle) | Toggle Loop on/off |
| Encoder 2 rotate (playing) | Adjust playback BPM |
| SW2 (Back) | Return to previous screen; does not stop playback or recording |

---

## Using Recorded Files in the Step Sequencer

A `.mid` file from the capture directory can be imported directly into the step sequencer's arranger. See the import workflow in the Pattern Editor: [Pattern Editor → SMF import](pattern-editor.html).

---

## Practical Workflow: Record and Loop

A fast way to capture a melodic idea and immediately loop it:

1. Load an instrument chain (e.g. FluidSynth piano, channel 1).
2. Open MIDI Recorder. Set Loop = **on**.
3. Tap **⬤ Start MIDI Recording**.
4. Play your melody on the keyboard.
5. Tap **■ Stop MIDI Recording** at the end of the phrase.
6. Immediately tap the new file → playback starts looping.
7. The piano chain keeps playing the loop while you layer more chains or adjust sounds.

---

## Practical Workflow: Play a DAW Export

If you have a MIDI arrangement from a DAW (Logic, Ableton, etc.) you want to play through Zynthian's engines:

1. Export the MIDI file from the DAW.
2. Copy the `.mid` file to `$ZYNTHIAN_MY_DATA_DIR/files/Midi/` via USB or SCP.
3. Load chains on the MIDI channels matching the DAW arrangement (e.g. channel 1 = piano, channel 10 = drums).
4. Open MIDI Recorder → "SD> User MIDI Tracks" section → tap the file.
5. The file plays back through all loaded chains simultaneously.

If BPM does not match, rotate encoder 2 while playing to nudge tempo, or set BPM in Admin → MIDI → Tempo before starting playback.

---

## What's Next

- [Pattern Editor](pattern-editor.html) — step sequencer with SMF import
- [Snapshots](snapshots.html) — save complete session state
- [MIDI Controllers](midi.html) — connecting live MIDI input
- [Admin & System](admin-guide.html) — global MIDI and transport settings

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_midi_recorder.py`, `zyngine/zynthian_state_manager.py`.*
