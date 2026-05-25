# Custom MIDI Channel Routing

**Goal:** Route each MIDI channel from the EMU Xboard to a dedicated Zynthian chain. Switching the keyboard's transmit channel switches the active instrument.
**Prerequisites:** Zynthian running. EMU Xboard connected via USB. Audio output confirmed working.
**Access:** SSH · Webconf · VNC

---

## Part 1 — Two Chains, Two Channels `[draft]`

Prove the concept: Chain 1 responds to MIDI channel 1 only, Chain 2 to channel 2 only. Switching the EMU between the two channels switches which engine plays — no overlap, no bleed.

### Step 1 — Verify controller is detected

SSH into Zynthian and confirm the EMU Xboard appears as a MIDI client:

```bash
ssh root@zynthian.local
aconnect -l
```

Expected: a line containing `E-MU` or `Xboard` in the client list:

```
client 32: 'E-MU Xboard' [type=kernel,card=X]
```

If the EMU does not appear, unplug and replug the USB cable, then run `aconnect -l` again.

### Step 2 — Verify MIDI port is enabled

Open `http://zynthian.local` → **MIDI** → **Ports**.

Find the EMU Xboard entry. Confirm it is enabled (checkbox checked / toggle on). If not, enable it and click **Save**.

### Step 3 — Add Chain 1

In VNC, the main screen shows any existing chain strips and a **+** button. Tap **+** to add a new chain.

- Select an engine — **FluidSynth** is a good starting point (piano/keyboard sounds).
- Select a bank and preset. Choose any piano or keyboard sound.

Zynthian assigns new chains to successive MIDI channels automatically. This is Chain 1 → MIDI channel 1.

### Step 4 — Add Chain 2

Tap **+** again to add a second chain.

- Select a different engine or a clearly different-sounding preset — an organ, pad, or bass sound works well.

This is Chain 2 → MIDI channel 2. Choose something easy to distinguish from Chain 1 by ear.

### Step 5 — Confirm channel assignments

Tap **Chain 1** in the main screen to open chain control. Navigate to the **Options** subscreen and find the **MIDI Channel** setting — it should read **1**.

Tap back, then tap **Chain 2** → **Options** → **MIDI Channel** — it should read **2**.

If either shows a different value or **Omni**, change it to the correct channel number.

### Step 6 — Switch EMU to channel 1 and play

On the EMU Xboard:

1. Press the **MIDI Channel** button. The MIDI Channel LED lights.
2. Use the data slider or **Octave +/-** buttons to select channel **1**.
3. Play notes.

Only Chain 1 should sound. Chain 2 should stay silent.

### Step 7 — Switch EMU to channel 2 and play

Press **MIDI Channel** on the EMU and select channel **2**. Play the same notes.

Only Chain 2 should sound now. Chain 1 should be silent.

**Verify:** Clear difference between the two channels. Switching channels on the keyboard switches the active instrument instantly with no overlap between chains.

---

## Part 2 — Scale to Four Channels `[draft]`

Add Chain 3 and Chain 4. By the end, channels 1–4 each trigger a distinct instrument with full channel isolation.

### Step 1 — Add Chain 3

Tap **+** → select an engine and preset. Zynthian assigns MIDI channel 3 automatically. Choose a sound distinct from Chains 1 and 2.

### Step 2 — Add Chain 4

Tap **+** → select an engine and preset. Assigned to MIDI channel 4. Again: choose a clearly distinct sound.

### Step 3 — Confirm channels 3 and 4

Tap each new chain → **Options** → **MIDI Channel** and confirm the assignment matches (3 and 4 respectively).

### Step 4 — Test all four channels

Cycle the EMU through channels 1, 2, 3, 4 using the **MIDI Channel** button. Play notes at each channel.

**Verify:** Each channel triggers exactly one chain. No other chains respond. Four distinct sounds, zero crosstalk.

---

## Part 3 — Save as Snapshot `[draft]`

Save the chain/channel map as a named snapshot and set it as the boot default so it loads automatically on every start.

### Step 1 — Save a named snapshot

Open `http://zynthian.local` → **Snapshots** → **Save As**.

Enter a name (e.g. `midi-channel-map`) and click **Save**.

### Step 2 — Set as boot default

In the snapshot list, find `midi-channel-map`. Click **Save as Last State**.

This writes the snapshot to `last_state.zss` — Zynthian loads this file automatically on every boot.

### Step 3 — Verify persistence

Reboot from SSH:

```bash
reboot
```

Wait ~30 seconds. Reconnect via VNC and confirm:
- All chains are present with their correct channel assignments
- EMU channel switching works immediately without any manual setup

**Verify:** After reboot, all chains load with the correct MIDI channel map. The setup is ready to play the moment Zynthian starts.

---

## Going Further

- **Layer mode** — set multiple chains to the same MIDI channel so one key press triggers multiple engines simultaneously
- **Note range splits** — restrict a chain to a key range so different keyboard zones trigger different instruments on a shared channel
- **SMC-PAD routing** — add a dedicated drum/percussion chain on channel 10, driven by the pad controller
- **Snapshot per setup** — create multiple named snapshots with different chain/channel maps, recall them via the webconf Snapshots page mid-session
