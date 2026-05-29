# Multi-Controller Performance Rig

**Goal:** Three controllers with distinct roles running simultaneously — Maschine MK2 + SMC-PAD drive a bass/trigger chain (ch1), E-MU Xboard drives a melodic pad chain (ch2), all knobs mapped to engine parameters, saved as the auto-loading boot state.
**Prerequisites:** Maschine MK2 daemon built and running (see [Maschine MK2 Controller](project-maschine-mk2.html)). SMC-PAD connected and active in webconf MIDI ports. Audio output working.
**Access:** SSH · Touchscreen · Webconf

---

## Part 1 — Dual-Chain MIDI Routing `[draft]`

Set Xboard to ch2, create two chains on separate channels, confirm each controller drives only its intended chain simultaneously.

### Step 1 — Understand the channel plan

All three controllers default to MIDI ch1. The Maschine MK2 daemon is hardcoded to ch1 and cannot be changed without modifying the source. The solution is to move the Xboard to ch2:

| Controller | MIDI channel | Drives |
|-----------|-------------|--------|
| Maschine MK2 | ch1 (fixed) | Chain 1 — bass / triggers |
| SMC-PAD pads | ch1 (default) | Chain 1 — harmonic triggers |
| E-MU Xboard | **ch2** (set manually) | Chain 2 — melodic pad |
| SMC-PAD knobs | ch1 | Chain 1 params |
| Xboard knobs | ch2 | Chain 2 params |

**Verify:** You understand the routing before continuing.

### Step 2 — Set Xboard to MIDI channel 2

On the E-MU Xboard:

1. Press the **MIDI Channel** button (labelled "10. MIDI Channel Select" in the manual).
2. Move the Data Slider or press **Octave Transpose +** / **−** to select channel **2**.
3. The display confirms the selected channel.

This setting is non-volatile — the Xboard remembers ch2 after power-off.

**Verify:** Press a key on the Xboard. Confirm via SSH that it sends on ch2:

```bash
ssh root@zynthian.local
amidi -p hw:$(aconnect -l | grep -i "Xboard\|EMU\|E-MU" | grep -oP 'client \K[0-9]+' | head -1),0,0 -d &
```

Press a key — the first byte of the note-on should be `91` (0x91 = note-on ch2). Press Ctrl+C.

**Verify:** Note-on messages show `91` as first byte (not `90` which is ch1).

### Step 3 — Start the Maschine MK2 daemon

If not already running:

```bash
ssh root@zynthian.local
systemctl status maschine-mk2.service --no-pager
```

If not active, start it:

```bash
systemctl start maschine-mk2.service
```

**Verify:** `aconnect -l` shows `maschine.rs` client.

### Step 4 — Add Chain 1: bass engine (ch1)

On the touchscreen, tap **+** → **Instrument** → **ZynAddSubFX**.

Browse to bank **A VDX** and select **Analog Bass** — a deep, sustained bass sound with clear low-end weight.

Set the chain to MIDI channel 1:
- Tap the chain → **Chain Options** → **MIDI Channel** → select **1**.

**Verify:** Chain 1 shows channel 1 in the chain view.

### Step 5 — Add Chain 2: melodic pad engine (ch2)

Tap **+** → **Instrument** → **ZynAddSubFX**.

Browse to bank **Cris Owl Alvarez** and select **ambient choirs** — a slowly evolving pad texture that works well over the bass.

Set the chain to MIDI channel 2: tap the chain → **Chain Options** → **MIDI Channel** → select **2**.

**Verify:** Chain 2 shows channel 2.

### Step 6 — Enable all MIDI ports in webconf

Open `http://zynthian.local` → **Interface → MIDI Options**. Click **MIDI Devices**. Enable:
- Xboard port
- `maschine.rs` port
- SMC-PAD port

**Verify:** All three ports toggled on.

### Step 7 — Test simultaneous routing

Press a Maschine MK2 pad — only Chain 1 (bass) responds.

Play a note on the Xboard — only Chain 2 (melodic pad) responds.

Both can play at the same time without interfering.

**Verify:** Each controller triggers its designated chain only. Both play simultaneously without either cutting the other off.

---

## Part 2 — SMC-PAD Integration and Knob Mapping `[draft]`

Route SMC-PAD pads to Chain 1, shift to bass register, map all knobs to engine parameters.

### Step 1 — Confirm SMC-PAD pads trigger Chain 1

SMC-PAD pads send notes on ch1 by default (Performance preset). Press any SMC-PAD pad — Chain 1 (bass) should respond.

**Verify:** SMC-PAD pads trigger Chain 1. This is correct — SMC-PAD pads and Maschine pads both drive the bass chain.

### Step 2 — Shift SMC-PAD pads to bass register

SMC-PAD default note range is mid-register. Shift down for a low bass register:

Press **Shift + Pad 15** twice to move down two octaves. Press **Shift + Pad 15 + Pad 16** to reset if needed.

**Verify:** SMC-PAD pads now produce bass-range notes, distinct from Maschine's note base.

### Step 3 — Set Maschine pads to low register

On the Maschine MK2, press **Group A** to activate the lowest note base (C2 or lower). Press Maschine pads to confirm they are in the bass register.

Group buttons A–H shift the note base upward. Group A = lowest.

**Verify:** Maschine pads and SMC-PAD pads cover complementary bass registers without overlapping.

### Step 4 — Map SMC-PAD knobs to Chain 1 parameters

On the touchscreen, tap Chain 1 to open its control screen.

For each parameter you want to control:
1. Long-press the parameter knob (~600ms) — turns orange.
2. Turn an SMC-PAD encoder (Knobs 1–8).

Suggested Chain 1 mapping (bass chain):

| SMC-PAD Knob | Parameter |
|---|---|
| 1 | Filter Cutoff |
| 2 | Filter Resonance |
| 3 | Amplitude / Volume |
| 4 | Amp LFO Depth |
| 5 | Filter LFO Freq |
| 6 | Portamento / Glide |
| 7 | Reverb Level |
| 8 | Pan |

**Verify:** Each SMC-PAD knob visibly moves the bound parameter when turned.

### Step 5 — Map Xboard knobs to Chain 2 parameters

Tap Chain 2 to open its control screen. Map Xboard CC knobs:

1. Long-press a parameter knob on the touchscreen (~600ms) — turns orange.
2. Turn a knob on the E-MU Xboard.

Xboard has 16 knobs — suggested Chain 2 mapping (melodic pad chain):

| Xboard Knob | Suggested Parameter |
|---|---|
| 1 | Filter Cutoff (CC 74) |
| 2 | Filter Resonance (CC 71) |
| 3 | Amplitude LFO Depth |
| 4 | Amplitude LFO Freq |
| 5 | Filter LFO Depth |
| 6 | Filter LFO Freq |
| 7 | Reverb / Effect Level |
| 8 | Volume (CC 7) |
| 9–16 | Additional ZynAddSubFX params as desired |

**Verify:** Xboard knobs control Chain 2 parameters without affecting Chain 1.

### Step 6 — Check SMC-PAD PLAY/STOP

Verify what PLAY/STOP send (from drone tutorial findings, or check again):

```bash
ssh root@zynthian.local
amidi -p hw:$(aconnect -l | grep -i "SMC\|SINCO" | grep -oP 'client \K[0-9]+' | head -1),0,0 -d &
```

Press PLAY, then STOP. Note output bytes. Press Ctrl+C.

If they send CC: map via CC Learn to a useful function (e.g. hold/release).
If they send MIDI real-time (FA/FC): note this for future use — no action needed now.

**Verify:** PLAY/STOP behaviour documented.

---

## Part 3 — Polish and Auto-Load `[draft]`

Enable Xboard aftertouch modulation, tune Maschine octave behaviour, set mixer levels, save as boot state.

### Step 1 — Enable Xboard aftertouch

Confirm aftertouch is active on the Xboard. From the manual:

> On the Xboard, Aftertouch transmits channel pressure messages when additional pressure is applied to a key after it is fully depressed.

Check current setting: press and hold a key on the Xboard, then press harder. If no aftertouch arrives, enable it:
- Press **Edit** → navigate to **Aft** (Aftertouch) → set to **On**.

**Verify:** Pressing harder on a held Xboard key sends additional MIDI data (visible as `Dn` byte in amidi monitor).

### Step 2 — Map aftertouch to modulation in Chain 2

Zynthian routes channel aftertouch automatically to CC 1 (modulation) on most engines. In ZynAddSubFX, modulation typically controls vibrato or filter modulation depth.

Play a sustained note on the Xboard and press harder — the timbre should change.

If no modulation occurs:
- In webconf → **Interface → MIDI Options** → **Midi filter rules**, add: `CH#2 ATAFTER => CH#2 CC#1` `[low]`.

**Verify:** Pressing harder on a held Xboard note produces an audible modulation effect.

### Step 3 — Set mixer levels for both chains

On the touchscreen, open the mixer (main screen → mixer view). Adjust:
- Chain 1 (bass) level: balance against Chain 2
- Chain 2 (melodic pad) level: leave some headroom

Pan Chain 1 slightly left and Chain 2 slightly right for width (optional).

**Verify:** Both chains sound balanced when playing together.

### Step 4 — Test the full rig

Play the complete setup together:
- Maschine MK2 pads: bass triggers on Chain 1
- SMC-PAD pads: harmonic bass triggers on Chain 1, knobs 1–8 shaping the bass tone
- Xboard: melodic pad on Chain 2, 16 knobs shaping the pad texture, aftertouch adding expression

**Verify:** All three controllers operate simultaneously without conflict. Each shapes its designated chain.

### Step 5 — Save as auto-loading boot state

Save the current state as the default startup snapshot:

```bash
ssh root@zynthian.local
cp /zynthian/zynthian-my-data/snapshots/$(ls -t /zynthian/zynthian-my-data/snapshots/ | head -1) /zynthian/zynthian-my-data/snapshots/last_state.zss
```

Or, go to **Library → Snapshots** → type `last_state` in the **Name:** field → click the checkmark button to save.

On next boot, Zynthian loads this snapshot automatically — both chains ready, all CC bindings restored, without any manual intervention.

**Verify:**

```bash
reboot
```

After ~45 seconds, SSH back in:

```bash
ssh root@zynthian.local
aconnect -l
```

**Verify:** `maschine.rs` is running (via systemd service), both chains are loaded, and the rig is ready to play immediately.

---

## Going Further

- Add a third chain on ch3 for a percussion engine (FluidSynth drum kit) triggered by Maschine Group B pads
- Use SMC-PAD KNOB BANK to switch to a second set of 8 knobs — total 16 SMC-PAD CC controls on Chain 1
- Add a MOD-UI effects chain after Chain 2 for reverb/delay on the melodic pad
- Map Maschine Group buttons A–H to trigger different bass registers — 8 octave zones across the 16 pads
- Use Xboard's 16-Channel Control Mode to send one CC across all 16 channels simultaneously for global parameter control
