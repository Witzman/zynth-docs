# Dub Techno Performance Loop

**Goal:** Build a looping dub techno foundation — a 16-step drum pattern and a sparse bass line playing simultaneously — as the basis for a live dub techno performance rig.
**Prerequisites:** Zynthian running and accessible via webconf and VNC. Audio output confirmed working (speakers or headphones connected to the ESI U46DJ mix output).
**Access:** Webconf · VNC

---

## Part 1 — Drums + Bass Loop `[draft]`

Set up two chains — a drum kit and a bass synth — each with a looping 16-step pattern, to prove the sequenced foundation works before adding effects or performance controls.

### Step 1 — Open the mixer screen

Connect to Zynthian via VNC. The main screen is the **Mixer** — it shows a row of chain strips (one per active sound source) with a **+** button at the bottom-left.

If you see a different screen, tap **Back** or **Home** until the mixer strip row is visible.

**Verify:** The mixer screen shows chain strips (or an empty mixer with only the **+** button visible). No other overlay or dialog is open.

### Step 2 — Add the drum chain

Tap **+** in the bottom-left of the mixer screen.

**Verify:** The **Add Chain...** screen appears, showing chain-type buttons.

### Step 3 — Select chain type for drums

Tap **Instrument**.

**Verify:** An engine list appears. You should see entries including **FluidSynth** and **ZynAddSubFX**.

> **Note:** Zynthian uses **Chains** (its term for a signal path from engine to audio output). All synth engines — including drum kits — use the **Instrument** chain type. There is no dedicated drum type.

### Step 4 — Select the FluidSynth engine

Tap **FluidSynth** in the engine list.

**Verify:** A bank list appears.

### Step 5 — Select the drum bank

Tap **System/FluidDrums** in the bank list.

**Verify:** A preset list appears showing drum kit presets.

### Step 6 — Select the Standard preset

Tap **Standard** in the preset list.

**Verify:** The chain control screen opens and a new chain strip labelled with the FluidSynth preset name appears in the mixer. You should be looking at the drum chain's parameter view.

### Step 7 — Return to the mixer

Tap **Back** to return to the mixer screen.

**Verify:** The mixer shows one chain strip for the drum kit. The strip is labelled with the preset or engine name.

### Step 8 — Switch to the Launcher

Tap the **Launcher** tab — this is the second tab in the mixer area, to the right of **Mixer**.

> **Note:** The **Launcher** is Zynthian's pattern grid — it shows a column of pattern pads for each chain. This is where you build and trigger step-sequencer patterns.

**Verify:** The Launcher view appears showing a grid. The drum chain has a column of pattern pads.

### Step 9 — Open the drum pattern pad

Tap the first pattern pad in the drum chain's column (top pad of the drum column).

**Verify:** The pad is selected or highlighted.

### Step 10 — Open the pattern editor

Tap the **Edit** button (pencil icon) in the toolbar, or long-press the pad and select **Edit** from the menu.

**Verify:** The Pattern Editor opens. You see a pitch/step grid — rows represent pitches (note names or numbers), columns represent steps.

### Step 11 — Set pattern length to 16 steps

In the Pattern Editor toolbar, find the **Length** control. Tap it and set the value to **16**.

**Verify:** The grid shows 16 step columns.

### Step 12 — Enter the kick drum

Find the row for **C2** (MIDI note 36 — Bass drum 1). In the pattern editor, rows are labelled by note name or number; C2 is MIDI note 36.

> **Note — if the pattern editor shows note numbers instead of names:** MIDI note 36 = C2 (kick), 38 = D2 (snare), 42 = F#2 (closed hi-hat). `[low]` — verify whether the editor displays note names or numbers on this Pi before publishing.

Tap steps **1, 5, 9, and 13** in the C2 row.

**Verify:** Four filled bars appear on the C2 row at steps 1, 5, 9, and 13. This places the kick on every beat of a 4/4 bar.

### Step 13 — Enter the snare

Find the row for **D2** (MIDI note 38 — Acoustic snare).

Tap steps **5 and 13**.

**Verify:** Two filled bars on the D2 row at steps 5 and 13. This places the snare on beats 2 and 4.

### Step 14 — Enter the closed hi-hat

Find the row for **F#2** (MIDI note 42 — Closed hi-hat).

Tap all **16 steps** in that row.

**Verify:** 16 filled bars on the F#2 row across all steps.

### Step 15 — Enable loop mode for the drum pattern `[low]`

> **Verify this step on the Pi before publishing.** The exact UI control for setting a pattern to loop vs. one-shot is unconfirmed. Look in the Pattern Editor toolbar for a **Loop** toggle, a **🔁** icon, or a mode selector. Enable whichever control sets this pattern to loop continuously.

Enable loop mode for the drum pattern.

**Verify:** The loop indicator is active (icon highlighted or toggle set to on). When played, the pattern will repeat automatically.

### Step 16 — Return to the mixer

Tap **Back** to return to the Launcher, then tap **Back** again (or the **Mixer** tab) to return to the mixer screen.

**Verify:** Mixer screen is visible. The drum chain strip is present.

### Step 17 — Add the bass chain

Tap **+** in the mixer screen.

**Verify:** The **Add Chain...** screen appears.

### Step 18 — Select chain type for bass

Tap **Instrument**.

**Verify:** Engine list appears.

### Step 19 — Select the ZynAddSubFX engine

Tap **ZynAddSubFX** in the engine list.

**Verify:** A bank list appears.

### Step 20 — Select the bass bank

Tap **A VDX** in the bank list.

**Verify:** A preset list appears.

### Step 21 — Select the Analog Bass preset

Tap **Analog Bass** in the preset list.

**Verify:** The chain control screen opens and a second chain strip appears in the mixer. The Analog Bass preset is a deep, sustained bass sound with clear low-end weight — appropriate for dub techno.

### Step 22 — Return to the mixer

Tap **Back** to return to the mixer.

**Verify:** Two chain strips are visible — drum kit and bass.

### Step 23 — Open the Launcher

Tap the **Launcher** tab.

**Verify:** Launcher view shows two columns — one for the drum chain, one for the bass chain.

### Step 24 — Open the bass pattern pad

Tap the first pattern pad in the bass chain's column.

**Verify:** The pad is selected or highlighted.

### Step 25 — Open the bass pattern editor

Tap **Edit** (pencil icon) in the toolbar, or long-press the pad and select **Edit**.

**Verify:** Pattern Editor opens for the bass chain.

### Step 26 — Set bass pattern length to 16 steps

In the Pattern Editor toolbar, set **Length** to **16**.

**Verify:** The grid shows 16 step columns.

### Step 27 — Enter the bass notes

Place **C2** (MIDI note 36) on steps **1, 3, 9, and 11**.

> **Note on pitch:** C2 is a good starting point — the low-end weight of the Analog Bass preset sits best in the C1–C3 range. The pattern is intentionally sparse; dub techno bass breathes in the gaps. Steps 1, 3, 9, 11 give a syncopated two-note-per-half-bar feel typical of dub techno.

**Verify:** Four filled bars on the C2 row at steps 1, 3, 9, and 11. Steps 2, 4, 5–8, 10, 12–16 are empty.

### Step 28 — Enable loop mode for the bass pattern `[low]`

> **Verify this step on the Pi before publishing.** Same as Step 15 — find and enable the loop control in the Pattern Editor toolbar for the bass pattern.

Enable loop mode.

**Verify:** Loop indicator is active for the bass pattern.

### Step 29 — Start transport `[low]`

> **Verify this step on the Pi before publishing.** The exact label and location of the Play/transport button in the Launcher or Pattern Editor toolbar is unconfirmed. Look for a **▶ Play** button or transport controls in the Launcher toolbar.

Tap **▶ Play** (or the equivalent transport start control) to start both patterns.

**Verify:** Both the drum and bass patterns begin playing. You hear the kick/snare/hi-hat pattern with the sparse bass notes layered underneath. The patterns loop continuously without drift. The sound is dry — no reverb or delay yet.

---

**Part 1 final verify:** Both chains are playing simultaneously. Drum pattern: kick on every beat, snare on beats 2 and 4, closed hi-hat on every step. Bass pattern: four notes per bar at steps 1, 3, 9, 11. Both patterns loop in sync with no drift. Audio is clean and dry.

---

### Step 30 — Stop transport

Tap **⏹ Stop** (or the equivalent transport stop control) to stop playback.

**Verify:** Both patterns stop. No sound.

### Step 31 — Save a snapshot

Open a browser and go to:

```
http://zynthian.local
```

Go to **Library → Snapshots**.

In the **Name:** field, type `dub-techno-p1`.

Tap the checkmark icon to save.

**Verify:** The snapshot `dub-techno-p1` appears in the snapshot list. Loading it will restore both chains and their patterns.

**Part 1 complete.** You have a working drums + bass foundation. Parts 2 and 3 will add dub-style effects and performance controls.
