# Generative Drone Synth

**Goal:** Build a self-evolving ambient drone using ZynAddSubFX — LFOs slowly mutate timbre without input, SMC-PAD pads shift root note, 8 knobs shape the texture in real time.
**Prerequisites:** Zynthian booted with audio output working (U46DJ or similar). For Parts 2–3: SMC-PAD connected via USB-C cable and visible in webconf MIDI ports. BLE is not used — connect via USB only.
**Access:** VNC · Webconf · SSH

---

## Part 1 — Patch: Self-Evolving Drone `[draft]`

Build the drone chain from scratch: load ZynAddSubFX with an ambient preset that has built-in LFO modulation, set it to monophonic, and confirm the timbre evolves without any input.

### Step 1 — Add a ZynAddSubFX chain

In VNC, tap **+** (bottom-left) → **Instrument** → **ZynAddSubFX**.

**Verify:** A ZynAddSubFX chain appears in the main screen.

### Step 2 — Load an ambient drone preset

Browse to bank **Cris Owl Alvarez** and select **ambient choirs**. This preset has built-in LFO modulation — timbre slowly shifts on a held note. `[low]` — if it sounds static on Step 4, try alternatives: **Alex J → Deep Cosmos**, **Alex J → Sweet Quiet Space**, or **Alex J → Vast Space Synth**.

**Verify:** ZynAddSubFX loads the preset.

### Step 3 — Set voice mode to monophonic

Monophonic mode ensures root note changes are clean with no overlapping drones. In VNC, tap the chain control screen. `[low]` Navigate to the MIDI or Part settings and set max voices to **1**.

**Verify:** Press two different notes in quick succession on the Xboard. The second note cuts the first cleanly — no overlap.

### Step 4 — Hold a note and listen

On the E-MU Xboard keyboard, press and hold a single note (e.g. middle C). Hold it for 10–20 seconds.

**Verify:** The timbre slowly shifts — filter opens and closes, volume breathes, or harmonics emerge and fade. This is the preset's built-in LFO modulation. If the sound is static, go to Step 5. If it evolves, Part 1 is complete.

### Step 5 — (If static) Increase LFO depth in VNC

If the held note sounds unchanging, navigate to the engine parameters in VNC:

1. Tap the chain control screen.
2. Navigate to the **Amplitude LFO** section.
3. Increase **Depth** to 40–60 and **Freq** to a slow value (0.3–0.8 Hz).
4. Navigate to **Filter LFO**.
5. Increase **Depth** to 30–50 and set **Freq** to a slightly different value from the amplitude LFO (e.g. 0.5 Hz) — the offset creates beating between the two modulations.

**Verify:** Holding a note now produces an audibly evolving texture.

---

## Part 2 — Control: SMC-PAD Pads and PLAY/STOP `[draft]`

Connect the SMC-PAD via USB, confirm it is routed, check what PLAY/STOP send, and map pads to root note changes.

### Step 1 — Connect the SMC-PAD via USB

Connect the SMC-PAD to the Raspberry Pi using a USB-C cable.

> **Note:** Bluetooth MIDI (BLE) is not used. Connect via USB only.

**Verify:** On the Pi, the device appears. Confirm via SSH:

```bash
ssh root@zynthian.local
aconnect -l | grep SINCO
```

Expected: three SINCO ports listed (SINCO SMC-PAD-Private, SINCO SMC-PAD-Master, SINCO).

### Step 2 — Enable the SMC-PAD port in webconf

The SMC-PAD appears as three MIDI ports in Zynthian. Only the main port needs to be enabled.

Open `http://zynthian.local` → **Interface → MIDI Options**. Click **MIDI Devices**. Find **SINCO 2** and enable it.

> **SINCO 2** is the SMC-PAD-Master port — this sends pad notes, knob CCs, and transport messages. SINCO 1 and SINCO 3 are secondary ports and can be left disabled.

**Verify:** SINCO 2 is toggled on.

### Step 3 — Check what PLAY/STOP buttons send

PLAY and STOP are DAW transport buttons. Their MIDI output must be confirmed before mapping — they may send CC messages (CC-Learnable) or MIDI real-time messages (not CC-Learnable).

In VNC, tap the ZynAddSubFX chain to open the parameter control screen. Long-press any parameter knob (~600ms) until it turns orange — CC Learn is now active.

Press the **PLAY** button on the SMC-PAD. Watch the knob:

| Result | What it means |
|--------|---------------|
| Knob returns to normal colour | PLAY sent a CC — Zynthian captured it. Go to Step 4A. |
| Knob stays orange | PLAY sent a MIDI real-time message (FA) — not CC-Learnable. Press the knob again to exit CC Learn. Go to Step 4B. |

Repeat with the **STOP** button.

**Verify:** You know whether PLAY/STOP send CC or real-time messages.

### Step 4A — If PLAY/STOP send CC: map sustain hold

If PLAY sends a CC, CC Learn already captured it. Now bind it to sustain:

Map the captured PLAY CC → value 127 (hold on) and the STOP CC → value 0 (hold off) via **Interface → MIDI Options** → **Midi filter rules** `[low]`. This makes PLAY hold a note indefinitely and STOP release it.

Skip to Step 5.

### Step 4B — If PLAY/STOP send real-time: use held pads

MIDI real-time messages (FA/FC) bypass the CC router and cannot be CC-Learned. Use held pads for drone start/stop instead:

Hold a pad to sustain a drone note. The note sounds for as long as the pad is pressed. When released, ZynAddSubFX's long release envelope fades the sound slowly rather than cutting it.

**Verify:** Pressing a pad starts the drone. Releasing causes a slow fade.

### Step 5 — Play pads as root note changes

SMC-PAD pads 1–16 send MIDI notes. Each pad is a different pitch. With monophonic mode active (Part 1 Step 4), pressing a pad silences the current drone and starts a new one on the new pitch.

Press pads 1–8 slowly in sequence. The drone shifts pitch with each press.

**Verify:** Each pad triggers a distinct drone pitch. The LFO texture continues evolving on the new pitch.

### Step 6 — Shift the octave range

SMC-PAD pads default to a mid-range note set. For a deeper drone, shift octaves down:

Press **Shift + Pad 15** to move pads down one octave. Repeat for a lower register. Press **Shift + Pad 15 + Pad 16** to reset to default. `[low]`

**Verify:** Pads produce lower pitches suitable for a bass drone.

---

## Part 3 — Shape: 8 Knobs to Parameters `[draft]`

Map the 8 SMC-PAD encoders to key ZynAddSubFX parameters for live textural control.

### Step 1 — Open the chain control screen in VNC

Tap the ZynAddSubFX chain to open its parameter control screen. Parameter knobs are visible — Filter Cutoff, Resonance, Amplitude, etc.

**Verify:** Control screen shows knobs for the engine parameters.

### Step 2 — Map Knob 1 → Filter Cutoff

Long-press the **Filter Cutoff** parameter knob in VNC (~600ms) until it turns orange. Turn **SMC-PAD Knob 1**. Zynthian captures the CC.

**Verify:** Turning Knob 1 now opens and closes the filter.

### Step 3 — Map remaining knobs

Repeat the long-press → turn knob process for each remaining parameter. Suggested mapping for a drone context:

| SMC-PAD Knob | Suggested Parameter | Effect |
|---|---|---|
| 1 | Filter Cutoff (CC 74) | Opens/closes texture brightness |
| 2 | Filter Resonance (CC 71) | Adds / removes filter peak |
| 3 | Amplitude LFO Depth | Breathing intensity |
| 4 | Amplitude LFO Freq | Breath speed |
| 5 | Filter LFO Depth | Filter sweep depth |
| 6 | Filter LFO Freq | Filter movement speed |
| 7 | Reverb / Effect Level | Space — wet/dry |
| 8 | Volume (CC 7) | Overall level |

Map each one: long-press target parameter in VNC → turn the SMC-PAD knob.

**Verify:** Each knob produces an audible change when turned while a drone note is held.

### Step 4 — Test the full setup

Hold a drone note (press and hold a pad, or use PLAY if it sends CC per Step 4A above). Slowly turn each knob and confirm the expected parameter changes. Press a different pad to shift root note. Confirm drone continues evolving.

**Verify:** All 8 knobs respond. Root note changes cleanly. LFO texture continues without interruption.

### Step 5 — Save final snapshot

In webconf, go to **Library → Snapshots**. Type `drone-final` in the **Name:** field and click the checkmark button to save. CC bindings are stored with the snapshot.

**Verify:** Snapshot saved. Reload it and confirm bindings survive.

---

## Going Further

- Use the KNOB BANK button on the SMC-PAD to switch to a second bank of 8 encoders — 16 total parameters under CC control
- Add a Calf Reverb or LSP Reverb LV2 effect chain after ZynAddSubFX for deeper space
- Add a second ZynAddSubFX chain on a different MIDI channel and use SMC-PAD Pad Bank 2 to play a harmony layer
- Explore ZynAddSubFX PAD synth module — "Bandwidth" and "Overtones" parameters create complex spectral drones
- Use Note Repeat on the SMC-PAD with Latch enabled to auto-repeat the held pad note at a slow rate
