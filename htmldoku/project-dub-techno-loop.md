# Dub Techno Performance Loop

**Goal:** Build a looping dub techno foundation — a 16-step drum pattern and a sparse bass line playing simultaneously — as the basis for a live dub techno performance rig.
**Prerequisites:** Zynthian running and accessible via touchscreen and webconf. Audio output confirmed working (speakers or headphones connected to the ESI U46DJ mix output).
**Access:** Touchscreen · Webconf

---

## Part 1 — Drums + Bass Loop `[draft]`

Set up two chains — a drum kit and a bass synth — each with a looping 16-step pattern, to prove the sequenced foundation works before adding effects or performance controls.

The two chains are pre-configured in a prepared snapshot. You load it, then build the patterns in the touchscreen step sequencer.

---

### Step 1 — Load the prepared snapshot

Tap **ZS3/SHOT** (bold hold, 300ms) → navigate into the **000** bank → select **dub-techno-p1**.

**Verify:** The mixer shows two chain strips — one for FluidSynth (drums) and one for ZynAddSubFX (bass). Both load automatically with the correct presets: **System/FluidDrums / Standard** and **A VDX / Analog Bass**.

---

### Step 2 — Open the Launcher

Tap **PAD/STEP** (short) on the V5 keypad.

> **Note:** The **Launcher** is Zynthian's step sequencer grid — columns = chains, rows = pattern slots. Each chain has its own column of pattern pads.

**Verify:** The Launcher view appears with two columns — one for the drum chain and one for the bass chain.

### Step 3 — Select the drum pattern pad

Tap the top pattern pad in the drum chain's column.

**Verify:** The pad is selected or highlighted.

### Step 4 — Open the drum pattern editor

Long-press the pad and tap **Edit**, or tap the pencil icon in the toolbar.

> **Note:** The Pattern Editor shows a pitch/step grid — rows = pitches, columns = steps. Each tap on an empty cell adds a note; double-tap removes it.

**Verify:** The Pattern Editor opens for the drum chain.

### Step 5 — Set pattern length to 16 steps

Tap **Length** in the toolbar and set the value to **16**.

**Verify:** The grid shows 16 step columns.

### Step 6 — Enter the kick drum

Find the row for **C2** (MIDI note 36 — Bass drum 1).

> **GM drum map reference:** C2 = kick drum (36) · D2 = acoustic snare (38) · F#2 = closed hi-hat (42). The editor labels rows by note name.

Tap steps **1, 5, 9, and 13** in the C2 row.

**Verify:** Four filled bars on C2 at steps 1, 5, 9, 13. Kick lands on every beat of a 4/4 bar.

### Step 7 — Enter the snare

Find the row for **D2** (MIDI note 38 — Acoustic snare).

Tap steps **5 and 13**.

**Verify:** Two filled bars on D2 at steps 5 and 13. Snare lands on beats 2 and 4.

### Step 8 — Enter the closed hi-hat

Find the row for **F#2** (MIDI note 42 — Closed hi-hat).

Tap all **16 steps** in that row.

**Verify:** 16 filled bars across the F#2 row — hi-hat on every 16th note.

### Step 9 — Enable loop mode for the drum pattern

Tap **🔁** (Loop) in the Pattern Editor toolbar.

**Verify:** The loop indicator is active. The pattern will repeat continuously when transport runs.

### Step 10 — Return to the Launcher

Tap **BACK/NO** on the V5 keypad.

**Verify:** Launcher view shows both chain columns. Drum pattern pad shows its filled state.

### Step 11 — Select the bass pattern pad

Tap the top pattern pad in the bass chain's column.

**Verify:** The pad is selected or highlighted.

### Step 12 — Open the bass pattern editor

Long-press the pad and tap **Edit**, or tap the pencil icon.

**Verify:** Pattern Editor opens for the bass chain.

### Step 13 — Set bass pattern length to 16 steps

Tap **Length** → set to **16**.

**Verify:** Grid shows 16 step columns.

### Step 14 — Enter the bass notes

Find the row for **C2** (MIDI note 36).

Tap steps **1, 3, 9, and 11**.

> **Note on pitch and pattern:** C2 sits in the weight range of the Analog Bass preset (best in C1–C3). The pattern is sparse — dub techno bass breathes in the gaps. Steps 1, 3, 9, 11 give a syncopated two-notes-per-half-bar feel typical of the genre.

**Verify:** Four filled bars on C2 at steps 1, 3, 9, 11. All other steps empty.

### Step 15 — Enable loop mode for the bass pattern

Tap **🔁** (Loop) in the Pattern Editor toolbar.

**Verify:** Loop indicator is active for the bass pattern.

### Step 16 — Return to the Launcher

Tap **BACK/NO** on the V5 keypad.

**Verify:** Launcher shows both chain columns with pattern pads filled.

### Step 17 — Start transport

Tap **PLAY (▶)** in the V5 keypad transport row (bottom of display).

**Verify:** Both patterns begin playing simultaneously. You hear kick/snare/hi-hat with sparse bass notes underneath. Patterns loop without drift. Sound is dry — no reverb or delay yet.

### Step 18 — Stop transport

Tap **STOP (■)** in the V5 keypad transport row.

**Verify:** Both patterns stop. No sound.

### Step 19 — Save the snapshot with patterns

Tap **OPT/ADMIN** (short) → **Snapshots** → navigate into the **000** bank → **Save as new snapshot** → type `dub-techno-p1` → confirm.

Navigate into the **000** bank folder first, then type the name and tap the checkmark icon. (Snapshots saved to the root level are invisible in the Zynthian UI.)

**Verify:** Snapshot `dub-techno-p1` appears in the list. The snapshot now includes both patterns.

---

**Verify:** Load `dub-techno-p1` from snapshots, press **PLAY (▶)** — both chains play immediately with no extra setup. Kick + snare + hi-hat driving, sparse bass underneath. Dry signal.

---

**Part 1 complete.** Two looping patterns in sync. Part 2 adds the pad layer and dub-style delay and reverb.
