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

## Part 2 — 8 Pages and Per-Step Note/Velocity `[draft]`

The sequencer has 8 independent 16-step pages. In sequencer mode, **Group A–H switch pages** — each button corresponds to one page. The lit Group button shows the active page.

Per-step note and velocity editing: tap any active step (its LED turns **orange** — selected), then turn Encoder 1 to adjust velocity (0–127) or Encoder 2 to adjust note offset (0–127).

Set the note base **before** entering sequencer mode: in pad mode, Group A–H set the note base (A = C1 through H = C8). Once you enter sequencer mode, those same buttons switch pages instead.

### Step 1 — Set note base in pad mode

Before entering sequencer mode, press **Group D** on the Maschine MK2 (while still in normal pad mode).

Group D sets the note base to **60 (C4)** — all steps with no note offset play C4.

**Verify:** Press a pad in pad mode — it plays C4 (requires a chain on MIDI ch 1 with a synth loaded).

### Step 2 — Enter sequencer mode

1. Hold **Shift**, press **Pad Mode** — enters pad mode 1
2. Hold **Shift**, press **Pad Mode** a second time — enters sequencer mode

**Verify:** Pressing a pad toggles a step (no note plays; LED brightens or dims).

### Step 3 — Navigate pages with Group buttons

In sequencer mode, pressing **Group A** switches to page 1, **Group B** to page 2, and so on. The lit Group button shows the active page.

1. Press **Group A** — page 1 is active. Program a short pattern: tap 3–4 pads to activate steps.
2. Press **Group B** — page 2 is active. Tap different pads to program a different pattern.
3. Press **Group A** again — the original page 1 pattern is still there.

**Verify:** Switching pages changes which step pads are lit. Each page holds its own independent pattern.

### Step 4 — Select a step for editing

Tap any lit (active) step pad. Its LED turns **orange** — this step is selected for per-step editing.

**Verify:** One pad glows orange while other active pads stay at normal brightness.

### Step 5 — Adjust velocity

With a step selected (orange LED), turn **Encoder 1** clockwise to raise velocity, counterclockwise to lower it.

Range: 0 (silent) to 127 (full velocity).

Press **Play**, then press **Erase** to stop. Listen for volume variation across steps.

**Verify:** The selected step plays at a noticeably different volume from unedited steps.

### Step 6 — Adjust note offset

With a step selected (orange LED), turn **Encoder 2** clockwise to raise the note offset, counterclockwise to lower it.

The note offset is added to the current note base. With Group D set before entering sequencer mode (base = 60 = C4):
- Offset 0 → C4
- Offset 7 → G4
- Offset 12 → C5

Press **Play** and listen for pitch variation across steps.

**Verify:** The selected step plays a different pitch from unedited steps.

### Step 7 — Build a phrase

Program a 4–8 step pattern. Select each step and assign different note offsets and velocities to create a melody.

**Verify:** Playback produces distinct pitches and volumes per step as programmed.

---

**Verify (Part 2 complete):** Pages are independent, Group buttons switch pages in sequencer mode, per-step velocity and note offset are audible on playback.

---

## Part 3 — Tempo and Snapshot `[draft]`

### Step 1 — Adjust tempo

[low] Hold **Shift** and turn **encoder B6** (second encoder in the top row) to change the step rate. Turn right to speed up, left to slow down. The README marks speed control as "under construction" — behavior needs Pi verification.

The default step rate is approximately 75 BPM in 16th notes (200 ms per step).

**Verify:** Turning the encoder changes the playback rate.

### Step 2 — Change note base between patterns

Group buttons A–H set the note base **in pad mode only**. In sequencer mode they switch pages. To shift the entire pattern's pitch range, exit sequencer mode first.

To change note base and return to sequencer mode:

1. Press **Erase** to stop playback
2. Hold **Shift**, press **Pad Mode** — returns to pad mode 1
3. Hold **Shift**, press **Pad Mode** again — returns to normal pad mode
4. Press a **Group** button to set note base (A = C1 … H = C8)
5. Re-enter sequencer mode: Shift + Pad Mode twice
6. Resume playback with **Play**

[low] Exact steps to navigate between pad mode and sequencer mode need Pi verification.

**Verify:** After setting a different Group button and re-entering sequencer mode, steps play at the new note base.

### Step 3 — Save the setup as a snapshot

Tap **OPT/ADMIN** (short) → **Snapshots** → navigate into **000** bank → **Save as new snapshot** → type name → confirm.

Navigate into the **000** bank folder first, then type the name and tap the checkmark icon. (Snapshots saved to the root level are invisible in the Zynthian UI.)

The snapshot saves the Zynthian chain configuration (engine, MIDI channel 2 assignment). It does not save the Maschine step pattern — that lives in the daemon's runtime state and resets when the daemon restarts.

**Verify:** Snapshot appears in the list and reloads the chain on channel 2.

---

## Part 4 — Euclidean Fill `[draft]`

Hold **Shift** and press any Group button (A–H) to fill the current sequencer page with an evenly-distributed rhythm. Group A = 1 hit, Group H = 8 hits. The hits are distributed using the Bresenham (Euclidean) algorithm — the first hit always falls on step 0.

Note: bare Group buttons (without Shift) switch pages — that is Part 2 behaviour. Shift+Group fills the current page without switching.

| Shift + Group | Hits | Approximate pattern across 16 steps |
|---|---|---|
| A | 1 | step 0 only |
| B | 2 | steps 0, 8 |
| C | 3 | steps 0, 5, 10 |
| D | 4 | steps 0, 4, 8, 12 |
| E | 5 | steps 0, 3, 6, 9, 12 |
| F | 6 | steps 0, 2, 5, 7, 10, 13 |
| G | 7 | steps 0, 2, 4, 6, 8, 10, 12 |
| H | 8 | steps 0, 2, 4, 6, 8, 10, 12, 14 |

[low] Exact step positions for each density need Pi verification — positions shown are approximate.

### Step 1 — Switch to an empty page

In sequencer mode, press **Group C** to switch to page 3. If you completed Part 2, pages 1 and 2 already have patterns — page 3 should be empty.

If page 3 has steps, tap each lit pad to deactivate it before proceeding.

**Verify:** No step pads are lit on page 3.

### Step 2 — Apply a 4-hit Euclidean fill

Hold **Shift** and press **Group D** (4 hits).

**Verify:** 4 step pads light up, evenly spaced across the 16 positions (steps 0, 4, 8, 12).

### Step 3 — Start playback

Press **Play**.

**Verify:** 4 evenly-spaced notes play in a repeating loop at the current step rate.

### Step 4 — Try different densities

Hold **Shift** + press **Group H** — fills page with 8 hits.

Hold **Shift** + press **Group A** — fills page with 1 hit (step 0 only).

**Verify:** Pad LEDs update immediately to show each new pattern.

---

**Verify (Part 4 complete):** Shift+Group fills page with correct number of evenly-spaced hits; LED display matches; plays correctly; switching densities mid-playback works.

---

## Part 5 — MIDI Clock Sync `[draft]`

Connect any MIDI clock source (Zynthian, a DAW, or a hardware sequencer) to the daemon's `MIDI Control` ALSA input port. The sequencer locks to the external clock: 24 pulses per quarter note (ppqn), 6 ticks = one 16th-note step. A MIDI Start message resets the sequencer to step 0 and begins playback; Stop halts it and preserves position. If no clock tick arrives for 500 ms, the sequencer falls back to its internal BPM timer automatically.

**Prerequisites:** Part 1 complete. A MIDI clock source available — Zynthian transport or a DAW connected to the same machine.

### Step 1 — Find port numbers

```bash
ssh root@192.168.2.123
aconnect -l | grep -A3 maschine
```

Look for:
```
client 28: 'maschine.rs' [...]
    0 'Pads MIDI   '
    1 'MIDI Control'
```

Note the client number (e.g. `28`). The `MIDI Control` port is `28:1`.

[low] Client number varies. Verify on Pi.

### Step 2 — Find the Zynthian MIDI clock output

[low] Zynthian MIDI clock output port name needs Pi verification. Run:

```bash
aconnect -l
```

Look for a Zynthian or JACK MIDI output port that sends MIDI clock. Common candidates: `ZynMidiRouter` or a port bridged via `a2jmidid`.

**Verify:** At least one candidate MIDI clock output port is visible in `aconnect -l` — note the client number and port number for use in Step 3.

### Step 3 — Connect the clock source

```bash
aconnect <clock-source-client>:<port> 28:1
```

Replace `<clock-source-client>:<port>` with the clock output found in Step 2, and `28` with the actual maschine.rs client number.

Verify the connection:
```bash
aconnect -l | grep -A6 maschine
```

The clock source should appear listed as a sender to the maschine.rs client.

**Verify:** Connection is visible in `aconnect -l`.

### Step 4 — Start the clock and verify sync

Start transport in Zynthian (or the clock source). Press **Play** on the Maschine MK2.

**Verify:** Steps advance in sync with the Zynthian tempo. Changing tempo in Zynthian changes the step rate on the Maschine within 1–2 steps.

### Step 5 — Stop and verify position hold

Stop transport in Zynthian.

**Verify:** The sequencer halts. The last active step pad stays lit — position is not reset to step 0 on stop.

### Step 6 — Test fallback to internal clock

With the clock connected and sequencer running, disconnect:

```bash
aconnect -d <clock-source-client>:<port> 28:1
```

Wait 2–3 seconds.

**Verify:** The sequencer does not stall or freeze. After approximately 500 ms, it continues stepping at the last estimated BPM on the internal timer.

---

**Verify (Part 5 complete):** Sequencer locks to external clock, stops on transport stop, falls back on clock silence.

---

## Going Further

- Keep a chain on channel 1 for live pad play and a chain on channel 2 for the step sequencer — different engines, different roles, simultaneously active
- Program a bass pattern on the sequencer while improvising a melody with the pads
- Pre-assign step notes via OSC: the daemon listens on `127.0.0.1:42434` for `/maschine/midi_note_base` messages — scriptable from the Pi
- Combine with the Multi-Controller Performance Rig tutorial to add the Xboard and SMC-PAD alongside the Maschine sequencer
- Use MIDI clock sync (Part 5) with Zynthian's transport to keep the Maschine sequencer in tempo without manual BPM matching
- Combine euclidean patterns on multiple pages for polyrhythmic structures — page 1: 4 hits, page 2: 3 hits, alternate playback
