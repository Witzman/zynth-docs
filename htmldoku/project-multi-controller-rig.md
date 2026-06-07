# Multi-Controller Performance Rig

**Goal:** Four simultaneous chains driven by three controllers — SMC-PAD triggers a live drum kit, the Maschine MK2 step sequencer drives a bass chain, the Xboard switches between a strings chain and a lead chain — saved as snapshot `rig-v1`.
**Prerequisites:** Zynthian running, audio output working. Maschine MK2 daemon running (`maschine-mk2.service`). SMC-PAD connected via USB. E-MU Xboard connected via USB. SMC-PAD ctrldev deployed (see [SMC-PAD Launcher Control](project-smc-pad-launcher.html) Part 4).
**Access:** SSH · Touchscreen · Webconf

---

## Channel map

| Controller | Mode | MIDI ch | Chain |
|---|---|---|---|
| SMC-PAD | Pads (Preset 1) | 6 | Drums |
| SMC-PAD | Transport (CC 27/28/29) | 1 | Zynthian transport — via ctrldev |
| SMC-PAD | Left/Right (CC 25/26) | 1 | Cycle drum kits — via ctrldev |
| Maschine MK2 | Step sequencer playback | 2 | Bass |
| Maschine MK2 | Live pads (normal mode) | 1 | — (reserved for future use) |
| E-MU Xboard | Selected channel | 3 | Strings |
| E-MU Xboard | Selected channel | 4 | Lead |

---

## Part 1 — Build the four chains `[draft]`

Create the four instrument chains with the correct MIDI channel assignments.

### Step 1 — Enable MIDI ports

Open `http://zynthian.local/ui-midi-options` → **MIDI Devices**. Enable all three controller ports:
- E-MU Xboard (`E-MU Xboard25` or `USB Device 0x41e:0x3f00`)
- SMC-PAD (`SINCO IN 2`)
- Maschine MK2 daemon (`maschine.rs` → `Pads MIDI`)

Click **Save**.

**Verify:** All three ports are toggled on.

### Step 2 — Turn off Single Active Channel

Open `http://zynthian.local/ui-midi-options`. Find **Single Active Channel** and set it to **Off**.

Click **Save**.

With this off, every chain responds to its own MIDI channel independently. Multiple chains can play simultaneously.

**Verify:** Single Active Channel is off.

### Step 3 — Add the Drums chain (ch 6)

On the touchscreen, tap **+** → **Instrument** → **FluidSynth**.

Select bank **System/FluidDrums** → preset **Standard**.

Tap the new chain strip → **Chain Options** → **MIDI Channel** → set to **6**.

**Verify:** Drums chain shows MIDI Channel = 6 in Chain Options.

### Step 4 — Add the Bass chain (ch 2)

Tap **+** → **Instrument** → **ZynAddSubFX**.

Browse to bank **A VDX** → select **Analog Bass**.

Tap the chain → **Chain Options** → **MIDI Channel** → set to **2**.

**Verify:** Bass chain shows MIDI Channel = 2.

### Step 5 — Add the Strings chain (ch 3)

Tap **+** → **Instrument** → **ZynAddSubFX**.

Browse to bank **A VDX** → select **Vangelis Strings 1**.

Tap the chain → **Chain Options** → **MIDI Channel** → set to **3**.

**Verify:** Strings chain shows MIDI Channel = 3.

### Step 6 — Add the Lead chain (ch 4)

Tap **+** → **Instrument** → **ZynAddSubFX**.

Browse to bank **A VDX** → select **Analog Lead**.

Tap the chain → **Chain Options** → **MIDI Channel** → set to **4**.

**Verify:** Lead chain shows MIDI Channel = 4.

---

## Part 2 — SMC-PAD: drums and transport `[draft]`

Wire the SMC-PAD pads to the drum chain and confirm the transport buttons control Zynthian playback.

### Step 1 — Confirm SMC-PAD ctrldev is active

Tap **OPT/ADMIN** (bold hold, 300ms) → **MIDI** → **MIDI Input Devices**.

Find **SINCO IN 2**. It should show **SINCO SMC-PAD** as its driver. If not, long-press it and select **SINCO SMC-PAD**.

**Verify:** SINCO IN 2 shows driver = SINCO SMC-PAD.

### Step 2 — Confirm master MIDI channel is disabled

Open `http://zynthian.local/ui-midi-options`. Find **Master MIDI Channel**. It must be set to **0** (disabled).

> **Why:** The master channel intercepts ALL events on that channel before they reach chains. If set to 6, SMC-PAD pad notes are consumed and the drum chain receives nothing.

If it shows any value other than 0, clear it and click **Save**.

**Verify:** Master MIDI Channel = 0.

### Step 3 — Test drum pads

Press any SMC-PAD pad. You should hear a drum sound immediately.

The pad-to-sound mapping for Preset 1 (select with **Shift + Pad 1**):

| Pads | Notes | Sounds |
|---|---|---|
| 1–4 | 36–39 | Bass drum, side stick, snare, hand clap |
| 5–8 | 40–43 | Electric snare, floor tom, hi-hat, floor tom |
| 9–12 | 44–47 | Pedal hi-hat, low tom, open hi-hat, low-mid tom |
| 13–16 | 48–51 | Hi tom, crash, high tom, ride |

**Verify:** Each pad produces a distinct drum sound.

### Step 4 — Test transport buttons

Press **PLAY** on the SMC-PAD. The Zynthian transport starts (same effect as tapping PLAY on the touchscreen).

Press **STOP** — transport stops.

Press **REC** — record mode toggles.

**Verify:** SMC-PAD transport buttons behave identically to the V5 touchscreen transport buttons.

### Step 5 — Test drum kit cycling

Press **Left** (→ left transport button) on the SMC-PAD. The drum kit changes to the previous FluidSynth program.

Press **Right** — advances to the next kit.

**Verify:** Drum kit changes with Left/Right. Press a pad after each change to hear the new kit.

---

## Part 3 — Maschine MK2: step sequencer to bass chain `[draft]`

The Maschine MK2 daemon outputs two MIDI streams: live pad presses on ch 1, and step sequencer playback on ch 2. Ch 2 is routed to the Bass chain.

### Step 1 — Confirm daemon is running

```bash
ssh root@192.168.2.123
systemctl status maschine-mk2.service --no-pager
```

Expected: `Active: active (running)`.

**Verify:** Daemon is running.

### Step 2 — Confirm Pads MIDI port is connected

```bash
aconnect -l | grep -A2 'maschine.rs'
```

Expected output contains `Connecting To: 128:0` — the Maschine daemon's Pads MIDI port is wired to ZynMidiRouter.

**Verify:** `Connecting To: 128:0` appears under `Pads MIDI`.

### Step 3 — Enter sequencer mode

On the Maschine MK2, press **Pad Mode** twice to enter sequencer mode (padmode 2). The pad LEDs change to show the current step pattern.

> In sequencer mode, pressing a pad toggles a step on or off — no note is sent to Zynthian. Playback is driven by the internal sequencer timer or external MIDI clock.

### Step 4 — Program a bass pattern

Press **Play** on the Maschine MK2 to start the sequencer.

Toggle pads to add steps. Each active step plays a note on ch 2 → Bass chain when the sequencer reaches it.

To stop: press **Erase** (acts as stop in sequencer mode).

**Verify:** Steps you toggle play back on the Bass chain. The bass sound plays in time.

### Step 5 — Return to live mode

Press **Pad Mode** once to exit sequencer mode. Pads now send live NoteOn messages on ch 1. Ch 1 has no chain assigned — pressing pads produces no sound (by design; ch 1 is reserved for future use such as a live improv chain).

**Verify:** Pressing pads in normal mode produces no output. Entering sequencer mode and pressing Play resumes bass playback.

---

## Part 4 — E-MU Xboard: instrument switching `[draft]`

The Xboard transmits on whichever MIDI channel you select. Switching channels switches which chain responds.

### Step 1 — Set Xboard to channel 3 (Strings)

On the E-MU Xboard:

1. Press the **MIDI Channel** button. The MIDI Channel LED illuminates.
2. Use **Octave Transpose +** / **−** or the data slider to select channel **3**.
3. Press **Enter** to confirm.

Play any key. Only the Strings chain should respond.

**Verify:** Strings (Vangelis Strings 1) plays. Drums and Bass are silent.

### Step 2 — Switch Xboard to channel 4 (Lead)

Press **MIDI Channel** → select channel **4** → **Enter**.

Play any key. Only the Lead chain should respond.

**Verify:** Lead (Analog Lead) plays. No other chains respond.

### Step 3 — Confirm simultaneous operation

With Xboard on ch 3 or ch 4:

- Press SMC-PAD pads — drums play.
- Start the Maschine sequencer — bass plays.
- Play the Xboard — strings or lead plays.

All three play simultaneously without interfering.

**Verify:** All three controllers sound their designated chains at the same time.

---

## Part 5 — Save as rig-v1 `[draft]`

Save the complete setup as a named snapshot.

### Step 1 — Save the snapshot

Tap **OPT/ADMIN** (short) → **Snapshots** → navigate into the **000** bank.

Tap the checkmark icon. Type `rig-v1`. Confirm.

**Verify:** `rig-v1` appears in the snapshot list under bank 000.

### Step 2 — Set as boot default

In the snapshot list, select `rig-v1`. Tap **Save as Last State**.

**Verify:** On next boot, all four chains load automatically with the correct channel assignments.

---

## Going Further

- Add a fifth chain on ch 1 for Maschine live pad improv — pads now trigger a second melodic instrument alongside the sequencer.
- Use the SMC-PAD's PAD BANK button to switch to a second pad bank and map it to a second drum kit or a different instrument on a spare channel.
- Assign the SMC-PAD's 8 left-column encoders to drum chain parameters via MIDI CC Learn — long-press a parameter in the chain control screen, then turn an encoder.
- Use Maschine Group buttons A–H to switch sequencer pages and run up to 8 different bass patterns per session.
- Add a MOD-UI effects chain after the Strings or Lead chain for reverb and delay.
