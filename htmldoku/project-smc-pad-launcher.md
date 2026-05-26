# SMC-PAD Launcher Control

**Goal:** Use all 16 SMC-PAD pads to trigger Zynthian sequencer slots — 4 phrases × 4 chains, one pad per slot.
**Prerequisites:** Zynthian running and accessible via SSH and webconf. SMC-PAD available via USB.
**Access:** SSH · Webconf · VNC

---

## Part 1 — Connect and discover pad notes `[draft]`

Before writing any MIDI mappings, identify which MIDI note each pad sends and which channel they use.

### Step 1 — Connect SMC-PAD via USB

Plug the SMC-PAD into any USB port on the Raspberry Pi. USB MIDI is class-compliant — no driver needed.

**Verify:** The power LED on the SMC-PAD illuminates.

### Step 2 — Confirm Zynthian detects the device

SSH into the Pi and run:

```bash
ssh root@zynthian.local
aconnect -l
```

Look for an entry containing **SMC** or similar name, listed as a MIDI client with at least one port.

Example output:
```
client 32: 'SMC-PAD' [type=kernel,card=4]
    0 'SMC-PAD MIDI 1  '
```

Note the **card number** (the digit after `hw:` in the port name — e.g. `4` above). You need it for Step 3.

**Verify:** SMC-PAD appears in `aconnect -l` output.

### Step 3 — Capture raw MIDI from the pads

Start a MIDI monitor on the SMC-PAD's port. Replace `X` with your card number from Step 2:

```bash
amidi -d -p hw:X,0,0
```

Press each pad one at a time — from **Pad 1** (bottom-left) to **Pad 16** (top-right). Each press prints one line showing three bytes in hex, for example:

```
96 24 64
```

The three values are:
- Byte 1: MIDI status byte — upper nibble = event type, lower nibble = channel. `96` = note-on on channel 7. `90` = note-on on channel 1. `99` = note-on on channel 10.
- Byte 2: Note number in hex. `24` hex = 36 decimal.
- Byte 3: Velocity.

Record the note number (byte 2) for each pad. Convert hex to decimal using:

```bash
printf '%d\n' 0xNN   # replace NN with the hex value
```

Fill in this table as you go — you will need it in Part 3:

| Pad | Label | MIDI note (decimal) |
|-----|-------|---------------------|
| 1  | bottom-left | |
| 2  | | |
| 3  | | |
| 4  | bottom-right | |
| 5  | | |
| 6  | | |
| 7  | | |
| 8  | | |
| 9  | | |
| 10 | | |
| 11 | | |
| 12 | | |
| 13 | top-left | |
| 14 | | |
| 15 | | |
| 16 | top-right | |

Also note the **MIDI channel** from byte 1. Lower nibble of the status byte = channel index (0-based). Add 1 to get the channel number. Example: `96` → lower nibble `6` → channel 7.

Press `Ctrl+C` to stop the monitor.

**Verify:** You have 16 different note numbers (one per pad) and one channel number that all pads share.

---

## Part 2 — Set up the launcher grid `[draft]`

Build a 4 chains × 4 phrases launcher layout in Zynthian and load a minimal test pattern in each slot.

### Step 1 — Open the chain manager in VNC

Open a VNC connection to `zynthian.local`. On the mixer screen, tap the **Chain Manager** icon (or go via **Main Menu → Chain Manager**).

**Verify:** Chain Manager screen is visible.

### Step 2 — Add 4 instrument chains

Tap **+** → **Instrument** → choose any available engine (e.g. **ZynAddSubFX**) → select any bank and preset. Repeat until you have 4 chains.

The chains will appear as columns in the launcher.

**Verify:** Mixer shows 4 chain strips side by side.

### Step 3 — Switch to launcher view

On the mixer screen, tap **Launcher** (the tab below the chain strips, to the right of **Mixer**).

**Verify:** Launcher view appears showing 1 row of 4 pads (one column per chain).

### Step 4 — Add 3 more phrase rows

Long-press on the phrase row label on the left side of the launcher. A menu appears. Select **Append phrase**. Repeat twice more until you have 4 phrase rows.

**Verify:** Launcher shows a 4 × 4 grid (4 rows, 4 columns).

### Step 5 — Add a test pattern to each slot

Tap any empty pad in the launcher to select it. Then tap the **Edit** button (pencil icon) or long-press the pad and choose **Edit**. The Pattern Editor opens. Add at least one note — tap any cell in the grid. Press **Back** to return to the launcher.

Repeat for all 16 slots — each slot needs at least one note so the sequencer has a pattern to play.

**Verify:** All 16 pads in the launcher show a non-empty state (colored, not blank).

---

## Part 3 — Map pads to sequencer slots `[draft]`

Configure Zynthian's master MIDI channel to match the SMC-PAD's pad channel, then add 16 note→CUIA mappings — one per slot.

### Step 1 — Open MIDI options in webconf

In a browser, open:

```
http://zynthian.local/ui-midi-options
```

Log in if prompted (default: admin/raspberry).

**Verify:** MIDI Options page loads.

### Step 2 — Set the master MIDI channel

Find the **Master MIDI Channel** field. Set it to the channel number you recorded in Part 1, Step 3.

Example: if byte 1 was `96` (channel 7), enter `7`.

**Verify:** Master MIDI Channel is set to your pad channel.

### Step 3 — Add the 16 TOGGLE_SEQ mappings

Find the **Master Key Actions** field. It contains a text area with existing note→action lines (one per line, format: `note_number: ACTION`).

Add 16 new lines using the note numbers you recorded in Part 1. Map each pad to its slot using the layout below:

```
Launcher layout (phrase × chain):

  Phrase 0  →  Pads 1–4   (bottom row)
  Phrase 1  →  Pads 5–8
  Phrase 2  →  Pads 9–12
  Phrase 3  →  Pads 13–16 (top row)

  Chain 0 = leftmost column, Chain 3 = rightmost column
```

The CUIA action is `TOGGLE_SEQ phrase,chain`. Example mapping block assuming pads send notes 36–51:

```
36: TOGGLE_SEQ 0,0
37: TOGGLE_SEQ 0,1
38: TOGGLE_SEQ 0,2
39: TOGGLE_SEQ 0,3
40: TOGGLE_SEQ 1,0
41: TOGGLE_SEQ 1,1
42: TOGGLE_SEQ 1,2
43: TOGGLE_SEQ 1,3
44: TOGGLE_SEQ 2,0
45: TOGGLE_SEQ 2,1
46: TOGGLE_SEQ 2,2
47: TOGGLE_SEQ 2,3
48: TOGGLE_SEQ 3,0
49: TOGGLE_SEQ 3,1
50: TOGGLE_SEQ 3,2
51: TOGGLE_SEQ 3,3
```

Replace `36`–`51` with your actual note numbers from Part 1.

Check the existing lines in the text area for conflicts — any note number already listed with a different action will be overridden by your new entry. Remove conflicting lines if needed.

**Verify:** 16 new lines appear in the text area with correct note numbers and TOGGLE_SEQ targets.

### Step 4 — Save and apply

Click **Save**. Zynthian applies the configuration without a full restart.

**Verify:** Page reloads with your entries still present.

### Step 5 — Test each pad

On the SMC-PAD, press each pad once. In the VNC launcher view, the corresponding slot should toggle between playing (active indicator) and stopped.

Press the same pad again — the slot should stop.

**Verify:** All 16 pads toggle their corresponding slots. No pad triggers the wrong slot.

---

## Going Further

- Add real patterns to each slot — the test patterns can be replaced with actual sequences.
- Use the SMC-PAD's PAD BANK button to switch to a second bank of 16 pads for a second scene.
- Assign the SMC-PAD's encoders to synth parameters via MIDI CC Learn.
- Save the full setup as a snapshot so the pad mappings load with a single recall.
