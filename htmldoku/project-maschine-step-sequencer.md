# Maschine MK2 Step Sequencer

**Goal:** Use the MaschineMK2_linux daemon's built-in 16-step sequencer to trigger Zynthian synth engines — no Native Instruments software required.
**Prerequisites:** [Maschine MK2 Controller](project-maschine-mk2.html) tutorial complete — daemon installed, systemd service active, JACK routing wired to ZynMidiRouter.
**Access:** SSH · Touchscreen · VNC

---

## Part 1 — First Sequence `[draft]`

The daemon's step sequencer outputs MIDI on **channel 2** — one channel above the normal pad output on channel 1. A chain set to MIDI channel 2 responds only to the sequencer; the pads (channel 1) leave it silent.

### Step 1 — Confirm the daemon is running

```bash
ssh root@zynthian.local
systemctl status maschine-mk2.service --no-pager
```

Expected: `Active: active (running)`

If not running:

```bash
systemctl start maschine-mk2.service
```

**Verify:** Service is active.

### Step 2 — Add a chain for the sequencer

Tap **+** at right edge of Mixer.

1. Tap **Instrument**
2. Choose an engine — **ZynAddSubFX** with any preset works for this test
3. Tap the new chain to open it, then open **Chain Options**
4. Set **MIDI Channel** to **2**

**Verify:** Chain appears in the mixer with an engine loaded and MIDI channel set to 2.

### Step 3 — Enter sequencer mode

On the Maschine MK2:

1. Hold **Shift**, press **Pad Mode** — enters pad mode 1
2. Hold **Shift**, press **Pad Mode** a second time — enters sequencer mode (pad mode 2)

In sequencer mode, pressing a pad does not play a note — it toggles that pad as an active step.

[low] Exact pad LED behavior on mode change needs Pi verification.

**Verify:** Pressing a pad produces no sound from Zynthian.

### Step 4 — Program a simple pattern

Tap four pads — for example, the first pad in each row (bottom-to-top). Each tap should light the pad up brighter to mark the step as active. Tap a lit pad to deactivate it.

**Verify:** Four pads are lit; the others are dim.

### Step 5 — Start playback

Press **Play** on the Maschine MK2.

**Verify:** Zynthian plays a repeating 4-note pattern at a steady tempo — each active step fires one note.

### Step 6 — Stop playback

Press the **Erase** button to stop.

The **Erase** button acts as Stop in sequencer mode. The **Play** button only starts.

**Verify:** Playback stops.

---

## Part 2 — Melodic Pattern `[draft]`

By default all 16 steps play C3 (MIDI 48). To build a melody, assign different notes to different steps. The 16 pads act as a note keyboard when Shift is held: each pad maps to a pitch based on its position plus the current note base set by the Group buttons.

Default note base (no Group button pressed): C3 (MIDI 48). Group D sets it to C4 (MIDI 60).

Pad layout with **Group D** selected (note base C4, MIDI 60):

```
[ C4  C#4  D4  D#4 ]   ← top row    (pads 0–3)
[ G#3  A3  A#3  B3 ]   ← row 3      (pads 4–7)
[ E3  F3  F#3  G3  ]   ← row 2      (pads 8–11)
[ C3  C#3  D3  D#3 ]   ← bottom row (pads 12–15)
```

### Step 1 — Set note base to C4

Press **Group D** on the Maschine MK2.

All unassigned steps now play C4 instead of C3.

**Verify:** Press Play — pattern plays C4.

### Step 2 — Assign a note to a step

To change the note on step 0 (top-left pad) to D4:

1. Hold **Shift**, tap **pad 0** — selects step 0 as the target for note assignment
2. Hold **Shift**, tap **pad 2** (top row, third from left, D4 with Group D) — assigns D4 to step 0

[low] Visual feedback for the selected-step state needs Pi verification.

Repeat for other steps. Use the note layout above to map pads to pitches. Any step can be assigned any note in the current two-octave range shown.

**Verify:** Press Play — step 0 plays D4, other steps play C4.

### Step 3 — Build a melody

Assign different notes across several active steps to create a phrase. Use the chromatic pad layout above as a reference.

**Verify:** The sequence plays distinct pitches in the programmed order.

---

## Part 3 — Tempo and Snapshot `[draft]`

### Step 1 — Adjust tempo

[low] Hold **Shift** and turn **encoder B6** (second encoder in the top row) to change the step rate. Turn right to speed up, left to slow down. The README marks speed control as "under construction" — behavior needs Pi verification.

The default step rate is approximately 75 BPM in 16th notes (200 ms per step).

**Verify:** Turning the encoder changes the playback rate.

### Step 2 — Transpose with Group buttons

Press any Group button (A–H) while the sequencer is running to shift the note base of all steps:

| Button | Note base | Register |
|--------|-----------|----------|
| A | C1 | sub-bass |
| B | C2 | bass |
| C | C3 (default) | mid-low |
| D | C4 (middle C) | mid |
| E | C5 | mid-high |
| F | C6 | high |
| G | C7 | very high |
| H | C8 | extreme high |

[low] Whether Group transpose applies immediately to the playing step or only from the next step needs Pi verification.

**Verify:** Group button press shifts the pitch of the entire pattern up or down.

### Step 3 — Save the setup as a snapshot

Tap **OPT/ADMIN** (short) → **Snapshots** → navigate into **000** bank → **Save as new snapshot** → type name → confirm.

Navigate into the **000** bank folder first, then type the name and tap the checkmark icon. (Snapshots saved to the root level are invisible in the Zynthian UI.)

The snapshot saves the Zynthian chain configuration (engine, MIDI channel 2 assignment). It does not save the Maschine step pattern — that lives in the daemon's runtime state and resets when the daemon restarts.

**Verify:** Snapshot appears in the list and reloads the chain on channel 2.

---

## Going Further

- Keep a chain on channel 1 for live pad play and a chain on channel 2 for the step sequencer — different engines, different roles, simultaneously active
- Program a bass pattern on the sequencer while improvising a melody with the pads
- Pre-assign step notes via OSC: the daemon listens on `127.0.0.1:42434` for `/maschine/midi_note_base` messages — scriptable from the Pi
- Combine with the Multi-Controller Performance Rig tutorial to add the Xboard and SMC-PAD alongside the Maschine sequencer
