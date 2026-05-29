# Live Looper with SooperLooper

**Goal:** Record synth chains and live U46DJ audio into SooperLooper loop slots. SMC-PAD transport buttons control record/play/overdub on the selected loop. SMC-PAD pads 1–6 trigger individual loop slots independently.

**Prerequisites:**
- Zynthian booted and reachable at `zynthian.local`
- U46DJ connected via USB and configured as audio device (see ESI U46DJ USB Audio Setup)
- E-MU Xboard connected via USB and detected
- SMC-PAD connected via USB and detected

**Access:** Touchscreen · Webconf · SSH

---

## Part 1 — First Loop `[draft]`

SooperLooper captures audio from a single ZynAddSubFX synth chain and loops it back. Controlled from the touchscreen — no hardware mapping yet.

### Step 1 — Start from a clean state

Open the main menu: tap **OPT/ADMIN** on the keypad.

Select **Admin** → **Clean All** to remove any existing chains.

**Verify:** The mixer screen shows no chain strips — only the **+** button.

### Step 2 — Add a ZynAddSubFX instrument chain

Tap **+** in the mixer screen.

Select **Instrument** from the Add Chain screen.

Select **ZynAddSubFX** from the engine list.

Navigate to any bank and load any preset. A pad or synth preset works well — something with sustained audio so the loop is easy to hear.

**Verify:** Play a few notes on the Xboard. Sound comes from the U46DJ Mix output (your monitors/headphones). The chain strip appears in the mixer.

### Step 3 — Add a SooperLooper chain

Tap **+** in the mixer screen again.

Select **Audio Input** from the Add Chain screen.

Select **SooperLooper** from the engine list.

**Verify:** A second chain strip labeled **SooperLooper** appears in the mixer. Tapping it opens the SooperLooper control screen, which shows a grid of labeled buttons (record, overdub, multiply, etc.) and a row of loop progress bars.

### Step 4 — Route the synth chain audio into SooperLooper

By default SooperLooper captures only physical audio inputs from the U46DJ. To loop synth audio, route the synth chain's output into SooperLooper.

Tap the **ZynAddSubFX** chain strip to open its chain control screen.

On the right side of the screen you will see the side chain panel — a small signal-flow graph showing nodes connected by lines.

Tap the **Audio Output** node at the bottom of the side chain graph. [low — verify node label on Pi]

The **Audio Out** subscreen opens. Under the **> Chain inputs** heading you will see a list of available destination chains with checkboxes.

Tap the **SooperLooper** entry to enable it (checkbox fills: ☑).

**Verify:** The SooperLooper chain is now listed as a selected destination in the Audio Out screen. Tap **Back** to return to the chain control. [low — verify routing takes effect without reboot]

### Step 5 — Record a loop

Tap the **SooperLooper** chain strip to open its control screen.

Tap the **record** button. The button highlights and the first loop progress bar begins filling — SooperLooper is recording.

Play a phrase on the Xboard (4–8 bars works well).

Tap **record** again to stop recording. SooperLooper immediately begins looping what you played.

**Verify:** You hear the recorded phrase looping continuously. The loop progress bar sweeps left to right in time with the loop. The **record** button returns to its normal color.

### Step 6 — Overdub a layer

While the loop plays, tap **overdub**. Play additional notes on the Xboard.

Tap **overdub** again to stop overdubbing. Both layers now loop together.

To undo the overdub: tap **undo**. The added layer disappears.

**Verify:** Undo removes only the last overdub; the original loop continues.

### Step 7 — Stop and clear

Tap the loop's progress bar to select it (if not already selected).

Tap **undo** repeatedly to remove all overdubs down to the original recording. Then, to clear the loop completely: [low — verify exact clear method — may require tapping loop canvas and then a clear action, or using undo_all from a menu]

**Verify:** The loop progress bar resets. SooperLooper is in the idle state, ready to record again.

---

## Part 2 — Add U46DJ Live Audio Input `[draft]`

Route U46DJ line/mic inputs into SooperLooper alongside the synth chain. Now loops can capture both played notes and live audio.

### Step 1 — Open SooperLooper's audio input routing

Tap the **SooperLooper** chain strip to open its control screen.

On the right side, in the side chain panel, tap the **Audio Input** node. [low — verify node label]

The **Audio In** subscreen opens. It lists the physical audio inputs available from the U46DJ.

### Step 2 — Enable U46DJ inputs

At 44.1 kHz, the U46DJ presents 4 capture channels: inputs 1–4.

Toggle on **Audio input 1** and **Audio input 2** (the phono/line L and R, or mic inputs — depending on your U46DJ input assignment). [low — verify exact input labels and channel numbers]

Tap **Back**.

**Verify:** Speak into a mic connected to U46DJ channel 1 or 2 while the SooperLooper input level meter is visible. The green bar in the **input level** display responds to the audio.

### Step 3 — Record a loop with live audio

Follow Part 1 Steps 5–6, but this time play the Xboard AND make sound through the U46DJ inputs simultaneously.

**Verify:** The recorded loop contains both the synth audio and the live audio from the U46DJ input.

---

## Part 3 — SMC-PAD Transport Binding `[draft]`

Map the SMC-PAD **RECORD**, **PLAY**, and **STOP** transport buttons to SooperLooper controls via MIDI filter rules and CC Learn.

The SMC-PAD transport buttons send Mackie Control MIDI messages — Note On events, not MIDI CC. Zynthian's MIDI filter rules can remap these Note On messages to CC so that CC Learn can bind them to SooperLooper controls.

**Mackie Control MIDI notes sent by SMC-PAD transport buttons** [low — verify on Pi with `amidi` before entering these rules]:

| Button | MIDI event | Note | Channel |
|--------|-----------|------|---------|
| RECORD | Note On | 95 | 1 (CH#0 in filter) |
| PLAY | Note On | 94 | 1 (CH#0 in filter) |
| STOP | Note On | 93 | 1 (CH#0 in filter) |

### Step 1 — Verify SMC-PAD transport MIDI output

Connect to Zynthian via SSH:

```bash
ssh root@zynthian.local
```

Find the SMC-PAD MIDI port:

```bash
aconnect -l
# → Look for a line like: client 32: 'SMC-PAD' [type=kernel,card=N]
```

Monitor raw MIDI from the SMC-PAD:

```bash
amidi -p hw:N,0,0 -d
# Replace N with the card number from aconnect output
```

Press **RECORD** on the SMC-PAD. Note the hex bytes printed. Expected: `90 5F 7F` (channel 1, Note On, note 95, velocity 127). [low — record actual values here during testing]

Press **PLAY** — expected: `90 5E 7F` (note 94). [low]

Press **STOP** — expected: `90 5D 7F` (note 93). [low]

Press **Ctrl+C** to stop monitoring.

### Step 2 — Add MIDI filter rules

Open `http://zynthian.local` → **Interface** → **MIDI Options**.

Scroll to **Midi filter rules**.

Add the following rules. Enter each rule on its own line.

Map RECORD (single_pedal — needs both press and release):

```
MAP CH#0 NON#95 => CH#0 CC#85
MAP CH#0 NOFF#95 => CH#0 CC#85
```

Map PLAY (trigger — press only):

```
MAP CH#0 NON#94 => CH#0 CC#86
```

Map STOP (pause — press only):

```
MAP CH#0 NON#93 => CH#0 CC#87
```

Click **Save** and then **Restart UI** to apply.

**Verify:**

```bash
amidi -p hw:N,0,0 -d
```

Press each transport button. The raw output should now show CC messages (`B0 55 7F`, `B0 56 7F`, `B0 57 7F`) instead of Note On events. [low — confirm with actual values]

### Step 3 — CC Learn: bind RECORD to single_pedal

`single_pedal` mode is an intelligent single-button looper control:
- Idle → starts recording (first press)
- Recording → stops recording and plays back (second press)
- Playing → overdubs (third press)
- Overdubbing → back to playing (fourth press)
- Double-tap: pause
- Triple-tap: clear loop

Tap the **SooperLooper** chain strip.

Navigate to the **Global loop** control page. [low — verify navigation: may need to swipe the control screen or tap F1/F2 to reach Global loop page]

Long-press the **single pedal** control (~600ms) until it highlights orange — CC Learn mode is active.

Press **RECORD** on the SMC-PAD.

**Verify:** The **single pedal** control shows the bound CC number (85). Pressing RECORD on the SMC-PAD now cycles through idle → record → play → overdub → play.

### Step 4 — CC Learn: bind PLAY to trigger

Navigate to the **Loop control** control page in SooperLooper.

Long-press the **trigger** control until it highlights orange.

Press **PLAY** on the SMC-PAD.

**Verify:** The **trigger** control shows CC 86. Pressing PLAY triggers the selected loop to play from the start.

### Step 5 — CC Learn: bind STOP to pause

Long-press the **pause** control until it highlights orange.

Press **STOP** on the SMC-PAD.

**Verify:** The **pause** control shows CC 87. Pressing STOP pauses the selected loop.

### Step 6 — Full transport test

Record a loop using RECORD: press once to start, press again to loop.

Press PLAY to trigger the loop. Press STOP to pause it. Press PLAY again to resume.

**Verify:** All three transport buttons control the loop without touching VNC.

---

## Part 4 — Pad-to-Loop Triggers `[draft]`

Map SMC-PAD pads 1–6 to trigger individual SooperLooper loop slots (loops 1–6 can be active simultaneously).

**SMC-PAD pad default MIDI notes (Performance preset, CH#0)** [low — verify before entering rules]:

| Pad | Note | Hex |
|-----|------|-----|
| 1 | 36 | 24 |
| 2 | 37 | 25 |
| 3 | 38 | 26 |
| 4 | 39 | 27 |
| 5 | 40 | 28 |
| 6 | 41 | 29 |

> **Note:** If the Xboard keyboard is also sending on MIDI channel 1, its notes in the C2–F2 range (notes 36–41) would also get remapped. Verify with `amidi` which channel the SMC-PAD pads use (it may differ from channel 1). Adjust `CH#0` in the rules below to match the pad channel if different.

### Step 1 — Verify SMC-PAD pad MIDI output

```bash
amidi -p hw:N,0,0 -d
```

Tap pad 1 on the SMC-PAD. Note the channel and note number. [low — record here during testing]

### Step 2 — Add pad MIDI filter rules

Open `http://zynthian.local` → **Interface** → **MIDI Options** → **Midi filter rules**.

Add these rules (adjust channel if pads do not use CH#0):

```
MAP CH#0 NON#36 => CH#0 CC#90
MAP CH#0 NON#37 => CH#0 CC#91
MAP CH#0 NON#38 => CH#0 CC#92
MAP CH#0 NON#39 => CH#0 CC#93
MAP CH#0 NON#40 => CH#0 CC#94
MAP CH#0 NON#41 => CH#0 CC#95
```

Click **Save** and **Restart UI**.

### Step 3 — Set SooperLooper to 6 loops

Tap the **SooperLooper** chain strip.

Navigate to the **Global loop** page.

Set **loop count** to 6. [low — verify control name and navigation]

Six loop progress bars appear in the SooperLooper widget.

### Step 4 — CC Learn pads to trigger:0 – trigger:5

In the SooperLooper control screen, navigate to the control page that shows individual loop triggers. [low — verify which page shows trigger:0–5 vs the selected-loop trigger]

For each loop slot:

1. Long-press **trigger (N)** until it highlights orange.
2. Tap the corresponding SMC-PAD pad (pad 1 for loop 1, etc.).
3. The control shows the bound CC number.

Repeat for all 6 loops (pads 1–6 → CC 90–95 → trigger:0–5).

### Step 5 — Test independent loop triggers

Record a loop on loop slot 1 (RECORD → play notes → RECORD).

Tap pad 1 → loop 1 triggers.

Add loop slot 2: in the SooperLooper widget, tap the second loop progress bar to select it, then record with RECORD.

Tap pad 2 → loop 2 triggers independently.

**Verify:** Pads 1 and 2 trigger their respective loops independently. Pressing one pad does not affect the other.

---

## Part 5 — Snapshot and Boot Persistence `[draft]`

Save the complete rig as a snapshot and set it to auto-load on boot.

### Step 1 — Save snapshot

Open `http://zynthian.local` → **Library** → **Snapshots**.

Type a name in the **Name:** field — for example, `live-looper`.

Click the checkmark icon to save.

### Step 2 — Set as boot default

Click **Save as Last State**.

**Verify:** Reboot Zynthian:

```bash
ssh root@zynthian.local
reboot
```

After boot, the SooperLooper and synth chains are present on the display, and the MIDI filter rules and CC bindings are active.

Test one transport button and one pad to confirm routing is intact.

---

## Going Further

- Add more instrument chains (FluidSynth bass, setBfree organ) and route each into SooperLooper for layered loops.
- Use the **overdub** button to layer multiple passes on a single loop slot.
- Assign SMC-PAD knobs 1–8 to SooperLooper **feedback**, **wet**, and **dry** controls via CC Learn — shape the decay of loops in real time.
- Use the **multiply** function (tap **multiply** while a loop plays) to extend the loop length by integer multiples.
- Save multiple snapshots for different performance setups and load them from webconf between songs.
