# Generative Drone Synth

**Goal:** Build a self-evolving ambient drone using ZynAddSubFX — LFOs slowly mutate timbre without input, SMC-PAD pads shift root note, 8 knobs shape the texture in real time.
**Prerequisites:** Zynthian booted with audio output working (U46DJ or similar). The `drone-v1` snapshot is pre-built on the Pi — no manual chain setup needed for Part 1. For Parts 2–3: SMC-PAD connected and visible in webconf MIDI ports.
**Access:** VNC · Webconf · SSH

---

## Part 1 — Patch: Self-Evolving Drone `[draft]`

The base configuration is pre-built in the `drone-v1` snapshot already on your Pi: a ZynAddSubFX chain named "Drone", Fantasy → Long SpaceChoir 1 preset, monophonic voice mode (one note at a time). Part 1 starts from that snapshot — load it, hold a note, and confirm the timbre evolves on its own.

### Step 1 — Load the drone-v1 snapshot

In webconf, go to **Library → Snapshots**. Find `drone-v1` in the list and click the load icon next to it.

**Verify:** In VNC, a chain labelled "Drone" appears, with ZynAddSubFX and "Long SpaceChoir1" visible.

### Step 2 — Hold a note and listen

On the E-MU Xboard keyboard, press and hold a single note (e.g. middle C). Hold it for 10–20 seconds.

**Verify:** The timbre slowly shifts — filter opens and closes, volume breathes, or harmonics emerge and fade. This is the preset's built-in LFO modulation. If the sound is static, go to Step 3. If it evolves, skip to Step 4.

### Step 3 — (If static) Increase LFO depth in VNC

If the held note sounds unchanging, navigate to the engine parameters in VNC:

1. Tap the chain control screen.
2. Navigate to the **Amplitude LFO** section.
3. Increase **Depth** to 40–60 and **Freq** to a slow value (0.3–0.8 Hz).
4. Navigate to **Filter LFO**.
5. Increase **Depth** to 30–50 and set **Freq** to a slightly different value from the amplitude LFO (e.g. 0.5 Hz) — the offset creates beating between the two modulations.

**Verify:** Holding a note now produces an audibly evolving texture.

### Step 4 — Confirm monophonic mode

The snapshot sets the engine to mono (one note at a time) — root note changes are clean with no overlapping drones.

Press two different notes in quick succession on the Xboard. The first should stop as the second sounds.

**Verify:** Only one note plays at a time. The second note immediately cuts the first.

---

## Part 2 — Control: SMC-PAD Pads and PLAY/STOP `[draft]`

Confirm SMC-PAD is routed, check what PLAY/STOP send, implement drone start/stop, and map pads to root note changes.

### Step 1 — Enable SMC-PAD in webconf

Open `http://zynthian.local` → **Interface → MIDI Options**. Click **MIDI Devices**. Find the SMC-PAD port and enable it.

**Verify:** SMC-PAD port is toggled on.

### Step 2 — Check what PLAY/STOP buttons send

The SMC-PAD manual states PLAY/STOP are DAW transport buttons — their exact MIDI output in Performance preset must be confirmed before mapping.

```bash
ssh root@zynthian.local
amidi -p hw:$(aconnect -l | grep -i "SMC\|SINCO" | grep -oP 'client \K[0-9]+' | head -1),0,0 -d &
```

This streams raw MIDI from the SMC-PAD. Now press the **PLAY** button, then the **STOP** button. Note the hex bytes that appear. Press Ctrl+C to stop.

**Possible outputs:**

| Output | What it means |
|--------|---------------|
| `FA` | MIDI Real-Time Start — not directly CC-Learnable |
| `FC` | MIDI Real-Time Stop — not directly CC-Learnable |
| `Bn XX YY` | CC message on channel n+1, CC number XX, value YY — CC-Learnable |

**Verify:** You can read the hex output for both PLAY and STOP.

### Step 3A — If PLAY/STOP send CC: map via CC Learn

If PLAY sends a CC (e.g. `B0 72 7F`):

In VNC, navigate to any parameter knob, long-press it (~600ms) until it turns orange, then press PLAY. Zynthian captures the CC.

Map PLAY → CC 64 value 127 (sustain hold on) and STOP → CC 64 value 0 (sustain off) via **Interface → MIDI Options** → **Midi filter rules** `[low]`. This makes PLAY hold the current note indefinitely and STOP release it.

Skip to Step 4.

### Step 3B — If PLAY/STOP send MIDI real-time: use two pads instead

MIDI real-time messages (FA/FC) bypass the MIDI CC router and cannot be CC-Learned in Zynthian. Use two SMC-PAD pads as start/stop instead:

- **Pad 13** (default note D#3): remap to send CC 64 value 127 via the SMC-PAD companion app — or use as a held note (sustain pedal equivalent)
- **Pad 14** (default note E3): remap to send CC 123 (All Notes Off)

If you do not have access to the SMC-PAD companion app (Android/iOS only):

Use Pad 13 normally — hold it to sustain a note. The note sustains for as long as you press the pad. ZynAddSubFX's long release envelope takes over when you release, fading slowly.

**Verify:** Pressing a pad starts the drone sound. Releasing causes a slow fade (not a cut — this is the LFO-driven release).

### Step 4 — Play pads as root note changes

In Performance preset, SMC-PAD pads 1–16 send MIDI notes. Each pad is a different pitch. With monophonic mode active (Part 1 Step 5), pressing a pad silences the current drone and starts a new one on the new pitch.

Press pads 1–8 slowly in sequence. The drone shifts pitch with each press.

**Verify:** Each pad triggers a distinct drone pitch. The LFO texture continues evolving on the new pitch.

### Step 5 — Shift the octave range

The SMC-PAD pads default to a mid-range note set. For a deeper drone, shift octaves down:

Press **Shift + Pad 15** to move pads down one octave. Repeat for a lower register. Press **Shift + Pad 15 + Pad 16** to reset to default.

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

Hold a drone note (pad or PLAY depending on Step 3A/3B above). Slowly turn each knob and confirm the expected parameter changes. Press a different pad to shift root note. Confirm drone continues evolving.

**Verify:** All 8 knobs respond. Root note changes cleanly. LFO texture continues without interruption.

### Step 5 — Save final snapshot

In webconf, go to **Library → Snapshots**. Type `drone-final` in the **Name:** field and click the checkmark button to save. CC bindings are stored with the snapshot.

**Verify:** Snapshot saved. Reload it and confirm bindings survive.

---

## Going Further

- Use the KNOB BANK button on the SMC-PAD to switch to a second set of 8 CC bindings — 16 total parameters under knob control
- Add a Calf Reverb or LSP Reverb LV2 effect chain after ZynAddSubFX for deeper space
- Add a second ZynAddSubFX chain on a different MIDI channel and use SMC-PAD Pad Bank 2 to play a harmony layer
- Explore ZynAddSubFX PAD synth module — "Bandwidth" and "Overtones" parameters create complex spectral drones
- Use Note Repeat on the SMC-PAD with Latch enabled to auto-repeat the held pad note at a slow rate
