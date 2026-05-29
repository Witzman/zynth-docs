# SMC-PAD Drum Computer Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Write and publish the SMC-PAD Drum Computer tutorial — 16 pads as a live GM drum kit + step-sequenced beat launcher.

**Architecture:** Single FluidSynth chain on the SMC-PAD's MIDI channel. Parts 1–3 drafted upfront with `[draft]` tags. Each part gets a `[verified]` tag only after the user confirms all steps pass on the Pi. Page added to the sidebar and committed after each status change.

**Tech Stack:** FluidSynth (GeneralUser GS soundfont), Zynthian pattern editor, webconf MIDI Options, `amidi` for verification. SMC-PAD Preset 1 (notes 36–51, channel 7).

---

## Files

| Action | Path |
|--------|------|
| Create | `htmldoku/project-drum-computer.md` |
| Modify | `htmldoku/generate-html.py` — add sidebar entry |
| Regenerate | `docs/zynthian-Doku/` — all HTML (generator rewrites every page's sidebar) |

---

## Task 1: Create tutorial file with header and Part 1

**Files:**
- Create: `htmldoku/project-drum-computer.md`

- [ ] **Step 1: Write the file**

Create `~/zynth-docs/htmldoku/project-drum-computer.md` with this exact content:

```markdown
# SMC-PAD Drum Computer

**Goal:** Use all 16 SMC-PAD pads as a drum computer — pads 1–12 trigger GM drum sounds live, pads 13–16 launch looped beat patterns.
**Prerequisites:** Zynthian running and accessible via SSH and webconf. SMC-PAD connected via USB and detected (verified in the [SMC-PAD Launcher Control](project-smc-pad-launcher.html) tutorial, Part 1).
**Access:** Webconf · VNC

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

### Step 1 — Add a FluidSynth drum chain

On the VNC desktop, tap **+** in the mixer screen. The **Add Chain...** screen appears. Tap **Instrument**.

Browse engines and select **FluidSynth**. The bank list appears.

Select the **GeneralUser GS** bank. [low — verify exact bank name on Pi]

A preset list appears. Select a drum kit preset — look for a name containing "Drums" or "Kit". [low — verify exact preset name on Pi]

**Verify:** A new chain strip appears in the mixer labelled with the FluidSynth preset name.

### Step 2 — Set the chain MIDI channel

Tap the new chain strip to open its control screen. Tap **Chain Options** in the sidebar.

Find **MIDI Channel** and set it to **7** (the channel SMC-PAD Preset 1 sends on).

Tap **Back** to return to the mixer.

**Verify:** Chain Options shows MIDI Channel = 7.

### Step 3 — Test live pad play

Press any pad on the SMC-PAD. You should hear the corresponding drum sound immediately.

Press pads 1–4 in sequence (bass drum, side stick, snare, hand clap).

**Verify:** Each pad produces a distinct drum sound. No notes are silent.

---

## Part 2 — Program a beat `[draft]`

Build a classic 4/4 drum pattern in the step sequencer, using the FluidSynth chain from Part 1.

### Step 1 — Switch to launcher view

On the mixer screen, tap the **Launcher** tab (to the right of **Mixer**, below the chain strips). [low — verify exact tab label]

**Verify:** Launcher view appears showing a grid of clip pads. The drum chain appears as a column.

### Step 2 — Open the pattern editor

Tap the clip pad for the drum chain (row 1, column 1 — the top-left pad in the launcher grid).

Tap the **Edit** button (pencil icon) in the toolbar, or long-press the pad and select **Edit**.

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

**Verify:** 16 bars on the F#2 row, all at lower velocity (visually dimmer or same size depending on UI).

### Step 7 — Play the pattern

In the Pattern Editor toolbar, tap **▶ Play**. The sequencer starts looping the 16-step pattern.

**Verify:** Kick on beats 1 and 3 (steps 1, 5, 9, 13), snare on beats 2 and 4 (steps 5, 13), closed hi-hat on every step. Pattern loops continuously.

### Step 8 — Stop and return to launcher

Tap **⏹ Stop** in the toolbar. Tap **Back** to return to the launcher view.

**Verify:** Back in launcher, the clip pad for the drum chain shows a non-empty state (colored, not blank).

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
- Build a variation — for example: row 2 = half-time beat, row 3 = fill, row 4 = breakdown
- Return to launcher

At minimum, add at least one note per slot so the sequencer has something to play.

**Verify:** All 4 clip pads in the drum chain column show a non-empty state.

### Step 3 — Set the master MIDI channel

In a browser, open:

```
http://zynthian.local/ui-midi-options
```

Find **Master MIDI Channel** and set it to **7** (the SMC-PAD's channel).

Click **Save**.

**Verify:** Page reloads with Master MIDI Channel = 7.

### Step 4 — Add TOGGLE_SEQ mappings for top-row pads

On the same MIDI Options page, find the **Master Key Actions** text area.

Add 4 lines — one per top-row pad:

```
48: TOGGLE_SEQ 0,0
49: TOGGLE_SEQ 0,1
50: TOGGLE_SEQ 0,2
51: TOGGLE_SEQ 0,3
```

Notes 48–51 are pads 13–16 (top row). `TOGGLE_SEQ phrase,chain` toggles phrase N of chain 0 (the drum chain is chain index 0).

Click **Save**.

**Verify:** 4 new lines appear in the text area. Page saves without error.

### Step 5 — Test launcher control

On the SMC-PAD, press pad 13 (top-left). In the launcher view, the phrase 1 clip should start playing (active indicator).

Press pad 13 again — the clip stops.

**Verify:** Pads 13–16 each toggle their corresponding phrase slot. Pads 1–12 still trigger live drum sounds.

> **Note:** If the top-row pads both play drum sounds AND toggle the sequencer, master key actions are passing notes through to the engine. This is valid — the hi tom and cymbal sounds will layer over the patterns. You can choose to leave this or remove notes 48–51 from the GM drum map step in Part 2.

### Step 6 — Save as snapshot

In a browser, open:

```
http://zynthian.local
```

Go to **Library → Snapshots**. In the **Name:** field enter `SMC-PAD Drum Computer`. Click the checkmark icon to save.

**Verify:** The snapshot appears in the list. Reloading it restores the drum chain, all patterns, and MIDI mappings.

---

## Going Further

- Replace the starter patterns with your own beats — the step editor supports velocity per note, swing via **Offset**, and probability via **Play Chance**.
- Use the SMC-PAD's PAD BANK button to switch to a second bank of 16 pads — map it to a second scene with different patterns.
- Assign the SMC-PAD's 8 encoders to synth parameters (reverb send, filter cutoff) via MIDI CC Learn — long-press a parameter on the chain control screen.
- Upload a custom `.sf2` soundfont via webconf → Presets to replace the GM kit with your own samples (LinuxSampler + SFZ is an alternative for SFZ-format sample packs).
```

- [ ] **Step 2: Commit the new file**

```bash
cd ~/zynth-docs
git add htmldoku/project-drum-computer.md
git commit -m "docs: add SMC-PAD Drum Computer tutorial (all parts draft)"
```

---

## Task 2: Add to sidebar and publish HTML

**Files:**
- Modify: `htmldoku/generate-html.py` line ~60 (Personal Projects section)
- Regenerate: `docs/zynthian-Doku/` (full directory)

- [ ] **Step 1: Add sidebar entry**

In `htmldoku/generate-html.py`, find the Personal Projects section (around line 54). Add one line after the `SMC-PAD Launcher Control` entry:

```python
        ("SMC-PAD Drum Computer", "project-drum-computer.html"),
```

The block should look like:

```python
    ("Personal Projects", [
        ("Personal MIDI Mapping", "project-midi-mapping.html"),
        ("Generative Drone Synth", "project-drone-synth.html"),
        ("Maschine MK2 Controller", "project-maschine-mk2.html"),
        ("MIDI Channel Routing", "project-midi-channel-routing.html"),
        ("EMU CC Knob Mapping", "project-emu-cc-learn.html"),
        ("SMC-PAD Launcher Control", "project-smc-pad-launcher.html"),
        ("SMC-PAD Drum Computer", "project-drum-computer.html"),
        ("ESI U46DJ Audio Setup", "project-u46dj-audio-setup.html"),
        ("Audio FX Chain with MOD-UI", "project-modui-effects.html"),
        ("Multi-Controller Performance Rig", "project-performance-rig.html"),
        ("Live Looper with SooperLooper", "project-live-looper.html"),
    ]),
```

- [ ] **Step 2: Run the generator**

```bash
cd ~/zynth-docs
python3 htmldoku/generate-html.py
```

Expected: no errors, generator prints page list including `project-drum-computer.html`.

- [ ] **Step 3: Commit everything**

```bash
cd ~/zynth-docs
git add htmldoku/generate-html.py docs/zynthian-Doku/
git commit -m "docs: add SMC-PAD Drum Computer to sidebar, regenerate HTML"
git push
```

- [ ] **Step 4: Update inwork.md**

Add entry to `MD/inwork.md` under the Tutorials section:

```
- [~] **SMC-PAD Drum Computer** — 16 pads as live GM drum kit + step-sequenced beat launcher; pads 13–16 launch patterns
```

```bash
cd ~/zynth-docs
git add MD/inwork.md
git commit -m "docs: track SMC-PAD Drum Computer tutorial in inwork"
git push
```

---

## Self-Review Notes

- **[low] tags:** Two names need Pi verification — exact FluidSynth bank name and drum preset name. Steps are written with `[low]` tags so they are not marked `[verified]` until confirmed.
- **TOGGLE_SEQ indices:** Phrase index is 0-based (0–3), chain index is 0 (first chain). Matches Zynthian source convention confirmed in SMC-PAD Launcher tutorial.
- **Note passthrough:** Step 5 of Part 3 includes an explicit note about the passthrough behaviour — user decides whether to leave it or not.
- **Spec coverage:** Part 1 (live play) ✓, Part 2 (step sequencer) ✓, Part 3 (launch from pads) ✓, Going Further (samples, bank switch, CC learn) ✓, snapshot strategy ✓.
