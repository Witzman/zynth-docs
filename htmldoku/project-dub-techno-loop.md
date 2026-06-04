# Dub Techno Performance Loop

**Goal:** Build a looping dub techno foundation — a 16-step drum pattern and a sparse bass line playing simultaneously — as the basis for a live dub techno performance rig.
**Prerequisites:** Zynthian running and accessible via webconf and VNC. Audio output confirmed working (speakers or headphones connected to the ESI U46DJ mix output).
**Access:** Webconf · VNC

---

## Part 1 — Drums + Bass Loop `[draft]`

Set up two chains — a drum kit and a bass synth — each with a looping 16-step pattern, to prove the sequenced foundation works before adding effects or performance controls.

> **V5 touch keypad:** This tutorial uses the V5 touch keypad (left-side panel). Key buttons used: **OPT/ADMIN** (short = Main Menu), **MIX/LEVEL** (short = Mixer), **PAD/STEP** (short = Launcher), **BACK/NO** (Back). Transport row at bottom: **PLAY (▶)**, **STOP (■)**, **REC (●)**.

### Step 1 — Open the mixer screen

Connect to Zynthian via VNC.

If you see a screen other than the **Mixer**, tap **MIX/LEVEL** (short) on the V5 keypad to jump there directly.

**Verify:** The Mixer screen shows chain strips (or an empty mixer). No overlay or dialog open.

### Step 2 — Open Chain Manager

Tap **OPT/ADMIN** (short tap) on the V5 keypad.

Tap **Chain Manager** in the Main Menu grid.

**Verify:** The Chain Manager screen appears — a visual graph of all active chains as columns.

### Step 3 — Add the drum chain

Tap **Add chain** in the Chain Manager.

**Verify:** The **Add Chain...** screen appears, showing chain-type buttons (Instrument, Audio Input, etc.).

### Step 4 — Select chain type for drums

Tap **Instrument**.

> **Note:** Zynthian uses **Chains** (its term for a signal path from engine to audio output). All synth engines — including drum kits — use the **Instrument** chain type. There is no dedicated drum type.

**Verify:** An engine list appears, including **FluidSynth** and **ZynAddSubFX**.

### Step 5 — Select the FluidSynth engine

Tap **FluidSynth**.

**Verify:** A bank list appears.

### Step 6 — Select the drum bank

Tap **System/FluidDrums**.

**Verify:** A preset list appears showing drum kit presets.

### Step 7 — Select the Standard preset

Tap **Standard**.

**Verify:** The chain control screen opens. The drum chain strip appears in the mixer.

### Step 8 — Return to the mixer

Tap **MIX/LEVEL** (short) on the V5 keypad.

**Verify:** Mixer shows one chain strip for the drum kit.

### Step 9 — Open the Launcher

Tap **PAD/STEP** (short) on the V5 keypad.

> **Note:** The **Launcher** is Zynthian's pattern grid — columns = chains, rows = pattern slots. This is where you build and trigger step-sequencer patterns.

**Verify:** The Launcher view appears. The drum chain has a column of pattern pads.

### Step 10 — Select the drum pattern pad

Tap the first pattern pad in the drum chain's column (top pad).

**Verify:** The pad is selected or highlighted.

### Step 11 — Open the pattern editor

Long-press the selected pad and tap **Edit**, or tap the pencil icon in the toolbar.

**Verify:** The Pattern Editor opens. A pitch/step grid appears — rows = pitches (note names), columns = steps.

### Step 12 — Set pattern length to 16 steps

Tap **Length** in the toolbar. Set the value to **16**.

**Verify:** The grid shows 16 step columns.

### Step 13 — Enter the kick drum

Find the row for **C2** (MIDI note 36 — Bass drum 1).

> **GM drum note reference:** C2 = kick (36), D2 = snare (38), F#2 = closed hi-hat (42). The editor shows note names by row.

Tap steps **1, 5, 9, and 13** in the C2 row.

**Verify:** Four filled bars on the C2 row at steps 1, 5, 9, 13. This places the kick on every beat of a 4/4 bar.

### Step 14 — Enter the snare

Find the row for **D2** (MIDI note 38 — Acoustic snare).

Tap steps **5 and 13**.

**Verify:** Two filled bars on the D2 row at steps 5 and 13 (beats 2 and 4).

### Step 15 — Enter the closed hi-hat

Find the row for **F#2** (MIDI note 42 — Closed hi-hat).

Tap all **16 steps** in that row.

**Verify:** 16 filled bars across the F#2 row.

### Step 16 — Enable loop mode

Tap **🔁** (Loop) in the Pattern Editor toolbar.

**Verify:** The loop indicator is active. The pattern will repeat continuously when transport runs.

### Step 17 — Return to mixer

Tap **MIX/LEVEL** (short) on the V5 keypad.

**Verify:** Mixer screen visible. Drum chain strip present.

### Step 18 — Add the bass chain

Tap **OPT/ADMIN** (short) → **Chain Manager** → **Add chain**.

**Verify:** The **Add Chain...** screen appears.

### Step 19 — Select chain type for bass

Tap **Instrument**.

**Verify:** Engine list appears.

### Step 20 — Select the ZynAddSubFX engine

Tap **ZynAddSubFX**.

**Verify:** A bank list appears.

### Step 21 — Select the bass bank

Tap **A VDX**.

**Verify:** A preset list appears.

### Step 22 — Select the Analog Bass preset

Tap **Analog Bass**.

> **Note:** Analog Bass is a deep, sustained bass sound with clear low-end weight — well suited to dub techno.

**Verify:** Chain control screen opens. A second chain strip appears in the mixer.

### Step 23 — Return to the Launcher

Tap **PAD/STEP** (short) on the V5 keypad.

**Verify:** Launcher shows two columns — drum chain and bass chain.

### Step 24 — Select the bass pattern pad

Tap the first pattern pad in the bass chain's column.

**Verify:** The pad is selected or highlighted.

### Step 25 — Open the bass pattern editor

Long-press the pad and tap **Edit**, or tap the pencil icon.

**Verify:** Pattern Editor opens for the bass chain.

### Step 26 — Set bass pattern length to 16 steps

Tap **Length** → set to **16**.

**Verify:** Grid shows 16 step columns.

### Step 27 — Enter the bass notes

Find the row for **C2** (MIDI note 36).

Tap steps **1, 3, 9, and 11**.

> **Note on pitch and pattern:** C2 sits in the low-end weight range of the Analog Bass preset. The pattern is intentionally sparse — dub techno bass breathes in the gaps. Steps 1, 3, 9, 11 give a syncopated feel typical of the genre.

**Verify:** Four filled bars on the C2 row at steps 1, 3, 9, 11. All other steps empty.

### Step 28 — Enable loop mode

Tap **🔁** (Loop) in the Pattern Editor toolbar.

**Verify:** Loop indicator is active for the bass pattern.

### Step 29 — Start transport

Tap **PLAY (▶)** in the V5 keypad transport row (bottom of display).

**Verify:** Both the drum and bass patterns begin playing simultaneously. You hear kick/snare/hi-hat with sparse bass notes underneath. Patterns loop continuously without drift. Sound is dry — no reverb or delay yet.

---

**Verify:** Both chains playing in sync. Drum pattern: kick every beat, snare beats 2 and 4, closed hi-hat every step. Bass: four notes at steps 1, 3, 9, 11. Clean, dry signal.

---

### Step 30 — Stop transport

Tap **STOP (■)** in the V5 keypad transport row.

**Verify:** Both patterns stop. No sound.

### Step 31 — Save a snapshot

In a browser, go to `http://zynthian.local` → **Library → Snapshots**.

In the **Name:** field, type `dub-techno-p1`. Tap the checkmark icon to save.

**Verify:** Snapshot `dub-techno-p1` appears in the list.

**Part 1 complete.** Drums + bass foundation is working. Part 2 adds the pad layer and dub effects.
