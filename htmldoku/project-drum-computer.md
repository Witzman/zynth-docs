# SMC-PAD Drum Computer

**Goal:** Use all 16 SMC-PAD pads as a drum computer — pads 1–12 trigger GM drum sounds live, pads 13–16 launch looped beat patterns.
**Prerequisites:** Zynthian running and accessible via touchscreen, webconf, and VNC. SMC-PAD connected via USB and detected (verified in the [SMC-PAD Launcher Control](project-smc-pad-launcher.html) tutorial, Part 1).
**Access:** Touchscreen · Webconf · VNC

---

## Part 1 — Live drum kit `[draft]`

Wire the SMC-PAD's 16 pads to a FluidSynth GM drumkit. Every pad press produces a drum sound immediately — no sequencer involved yet.

SMC-PAD Preset 1 sends notes 36–51 on channel 7. Those notes map directly onto the GM drum map:

| Pad | Note | Drum sound |
|-----|------|-----------|
| 1 | 36 (C2) | Bass drum 1 |
| 2 | 37 (C#2) | Side stick |
| 3 | 38 (D2) | Acoustic snare |
| 4 | 39 (D#2) | Hand clap |
| 5 | 40 (E2) | Electric snare |
| 6 | 41 (F2) | Low floor tom |
| 7 | 42 (F#2) | Closed hi-hat |
| 8 | 43 (G2) | High floor tom |
| 9 | 44 (G#2) | Pedal hi-hat |
| 10 | 45 (A2) | Low tom |
| 11 | 46 (A#2) | Open hi-hat |
| 12 | 47 (B2) | Low-mid tom |
| 13 | 48 (C3) | Hi tom |
| 14 | 49 (C#3) | Crash cymbal 1 |
| 15 | 50 (D3) | High tom |
| 16 | 51 (D#3) | Ride cymbal 1 |

If you are on a different SMC-PAD preset, verify your notes and channel first — follow Part 1 of the [SMC-PAD Launcher Control](project-smc-pad-launcher.html) tutorial.

> **Shortcut:** A pre-built snapshot named **SMC-PAD Drum Computer** is already saved in the Zynthian snapshot library (bank 000). Tap **ZS3/SHOT** (bold hold) → navigate into the **000** bank → select **SMC-PAD Drum Computer** to skip Steps 1–2 and go straight to Step 3.

### Step 1 — Add a FluidSynth drum chain

Tap **+** at the right edge of the Mixer. The **Add Chain...** screen appears. Tap **Instrument**.

Browse engines and select **FluidSynth**. The bank list appears.

Select the **System/FluidDrums** bank.

A preset list appears. Select **Standard**.

**Verify:** A new chain strip appears in the mixer labelled with the FluidSynth preset name.

### Step 2 — Set the chain MIDI channel

Tap the new chain strip in the Mixer to open its control screen. Tap **Chain Options** in the sidebar.

Find **MIDI Channel** and set it to **7** (the channel SMC-PAD Preset 1 sends on).

Tap **Back** to return to the mixer.

**Verify:** Chain Options shows MIDI Channel = 7.

### Step 3 — Test live pad play

Press any pad on the SMC-PAD. You should hear the corresponding drum sound immediately.

Press pads 1–4 in sequence (bass drum, side stick, snare, hand clap).

**Verify:** Each pad produces a distinct drum sound. No pads are silent.

---

## Part 2 — Program a beat `[draft]`

Build a classic 4/4 drum pattern in the step sequencer, using the FluidSynth chain from Part 1.

### Step 1 — Switch to launcher view

Tap **PAD/STEP** (short) to switch to the Launcher view. [low — verify exact tab label on Pi]

**Verify:** Launcher view appears showing a grid of clip pads. The drum chain appears as a column.

### Step 2 — Open the pattern editor

Tap the clip pad for the drum chain (row 1, column 1 — the top-left pad in the launcher grid).

Tap **PAD/STEP** (bold hold, 300ms) to open the Pattern Editor, or long-press the pad and select **Edit**.

**Verify:** Pattern Editor opens showing a pitch/step grid.

### Step 3 — Set pattern length to 16 steps

In the Pattern Editor toolbar, find the **Length** control. Tap it and set the value to **16**.

**Verify:** The grid shows 16 step columns.

### Step 4 — Enter the kick drum

In the notes grid, scroll until you find row **C2** (MIDI note 36 — Bass drum 1). Tap steps 1, 5, 9, and 13 in that row.

**Verify:** Four filled bars appear on the C2 row at steps 1, 5, 9, 13.

### Step 5 — Enter the snare

Find row **D2** (MIDI note 38 — Acoustic snare). Tap steps 5 and 13.

**Verify:** Two bars on the D2 row at steps 5 and 13.

### Step 6 — Enter the hi-hat

Find row **F#2** (MIDI note 42 — Closed hi-hat). Tap all 16 steps.

Long-press one of the hi-hat notes and set **Velocity** to **60** — quieter than kick and snare.

**Verify:** 16 bars on the F#2 row at lower velocity.

### Step 7 — Play the pattern

Tap **PLAY (▶)** in the bottom row. The sequencer starts looping the 16-step pattern.

**Verify:** Kick on beats 1 and 3 (steps 1, 5, 9, 13), snare on beats 2 and 4 (steps 5, 13), closed hi-hat on every step. Pattern loops continuously.

### Step 8 — Stop and return to launcher

Tap **STOP (■)** in the bottom row. Tap **BACK/NO** to return to the launcher view.

**Verify:** Back in the launcher, the clip pad for the drum chain shows a non-empty state (colored, not blank).

---

## Part 3 — Launch patterns from pads `[draft]`

Add 3 more phrases (patterns) to the drum chain, then map the SMC-PAD top row (pads 13–16) to launch each phrase. Pads 1–12 remain as live drum hits.

### Step 1 — Add 3 more phrase rows

In the launcher view, long-press the phrase row label on the left side of the grid. A menu appears. Select **Append phrase**. Repeat twice more.

**Verify:** Launcher shows 4 phrase rows for the drum chain.

### Step 2 — Add a pattern to each new phrase

For each of the 3 new phrase slots (rows 2, 3, 4):

- Tap the clip pad to select it
- Tap **Edit** (pencil icon)
- Build a variation — for example: row 2 = half-time feel (kick on steps 1 and 9 only), row 3 = fill (kick on steps 13–16), row 4 = breakdown (hi-hat only)
- Tap **Back** to return to the launcher

At minimum, add at least one note per slot.

**Verify:** All 4 clip pads in the drum chain column show a non-empty state.

### Step 3 — Verify master MIDI channel and key actions

The master MIDI channel and TOGGLE_SEQ pad mappings are already configured on this system. Confirm via webconf:

```
http://zynthian.local/ui-midi-options
```

Check that **Master MIDI Channel** shows **7** and **Master Key Actions** contains:

```
48: TOGGLE_SEQ 0,0
49: TOGGLE_SEQ 0,1
50: TOGGLE_SEQ 0,2
51: TOGGLE_SEQ 0,3
```

If either is missing, add or correct the values and click **Save**.

**Verify:** Master MIDI Channel = 7 and 4 TOGGLE_SEQ lines are present.

### Step 5 — Test launcher control

Press pad 13 (top-left) on the SMC-PAD. In the launcher view, the phrase 1 clip should start playing (active indicator).

Press pad 13 again — the clip stops.

Test pads 14, 15, and 16 — each should toggle its corresponding phrase slot.

**Verify:** Pads 13–16 each toggle their corresponding phrase slot. Pads 1–12 still trigger live drum sounds.

> **Note:** If the top-row pads both play drum sounds and toggle the sequencer simultaneously, master key actions are passing notes through to the engine. This is harmless — the hi tom and cymbal sounds layer over the patterns. Leave it or remove notes 48–51 from the hi-hat step in Part 2 if you prefer silence on those pads.

### Step 6 — Save as snapshot

Tap **OPT/ADMIN** (short) → **Snapshots** → navigate into the **000** bank → **Save as new snapshot** → type `SMC-PAD Drum Computer` → confirm.

> **Important:** Navigate into the **000** bank folder first, then type the name and tap the checkmark icon. Snapshots saved to the root level are invisible in the Zynthian UI.

**Verify:** The snapshot appears in the list. Loading it restores the drum chain, all 4 patterns, and MIDI mappings.

---

## Going Further

- Replace the starter patterns with your own beats — the step editor supports per-note velocity, swing via **Offset**, and probability via **Play Chance**.
- Use the SMC-PAD's PAD BANK button to switch to a second bank of 16 pads and map it to a second scene with different patterns.
- Assign the SMC-PAD's 8 encoders to synth parameters (reverb send, filter cutoff) via MIDI CC Learn — long-press a parameter on the chain control screen, then move an encoder.
- Upload a custom `.sf2` soundfont via webconf → Presets to replace the GM kit with your own samples. For SFZ-format sample packs, use LinuxSampler or Sfizz as the engine instead of FluidSynth.
