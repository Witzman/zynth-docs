# SMC-PAD Launcher Control

**Goal:** Use all 16 SMC-PAD pads to trigger Zynthian sequencer slots — 4 phrases × 4 chains, one pad per slot.
**Prerequisites:** Zynthian running and accessible via SSH and webconf. SMC-PAD available via USB.
**Access:** SSH · Webconf · Touchscreen

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

Look for an entry named **SINCO** with three ports:

```
client 28: 'SINCO' [type=kernel,card=3]
    0 'SINCO SMC-PAD-Private'
    1 'SINCO SMC-PAD-Master'
    2 'SINCO           '
```

Note the **card number** — the value of `card=N` on the SINCO line. Card numbers are assigned at boot and may differ each session.

**Verify:** SINCO appears in `aconnect -l` with three ports.

### Step 3 — Capture raw MIDI from the pads

Start a MIDI monitor on the SMC-PAD pad port. Replace `X` with your card number from Step 2:

```bash
amidi -d -p hw:X,0,1
```

Port `0,1` is the **SMC-PAD-Master** port — this is where pad notes, encoder CCs, and transport buttons are sent. Port `0,0` (Private) carries internal device messages and will not show pad presses.

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

The table below shows the verified note numbers for **Preset 1** (Performance preset — select with **Shift + Pad 1**). If you are using a different preset, fill in your own values here.

| Pad | Position | MIDI note (decimal) |
|-----|----------|---------------------|
| 1  | bottom-left  | 36 |
| 2  | bottom       | 37 |
| 3  | bottom       | 38 |
| 4  | bottom-right | 39 |
| 5  | left         | 40 |
| 6  |              | 41 |
| 7  |              | 42 |
| 8  | right        | 43 |
| 9  | left         | 44 |
| 10 |              | 45 |
| 11 |              | 46 |
| 12 | right        | 47 |
| 13 | top-left     | 48 |
| 14 | top          | 49 |
| 15 | top          | 50 |
| 16 | top-right    | 51 |

Also note the **MIDI channel** from byte 1. Lower nibble of the status byte = channel index (0-based). Add 1 to get the channel number. Example: `96` → lower nibble `6` → channel 7.

Press `Ctrl+C` to stop the monitor.

**Verify:** You have 16 different note numbers (one per pad) and one channel number that all pads share.

### Step 4 — Note encoder CC numbers

The 8 encoders send MIDI CC messages on the same port. The verified CC assignments for **Preset 1** are:

| Physical position | Knob # | CC |
|-------------------|--------|----|
| 1 (top-left)      | 7      | 16 |
| 2                 | 5      | 17 |
| 3                 | 3      | 18 |
| 4 (top-right)     | 1      | 30 |
| 5 (bottom-left)   | 8      | 80 |
| 6                 | 6      | 81 |
| 7                 | 4      | 82 |
| 8 (bottom-right)  | 2      | 31 |

Knob numbers are the device's internal numbering. Physical positions run left to right, top row then bottom row.

Use these CC numbers with Zynthian's MIDI CC Learn (long-press a parameter in the chain control screen) to assign encoders to synth parameters.

**Verify:** Turning each encoder produces a CC message in `amidi -d` output with the CC number listed above.

---

## Part 2 — Set up the launcher grid `[draft]`

Build a 4 chains × 4 phrases launcher layout in Zynthian and load a minimal test pattern in each slot.

### Step 1 — Open the chain manager

On the touchscreen mixer screen, tap the **Chain Manager** icon (or tap **OPT/ADMIN** on the keypad → **Main Menu → Chain Manager**).

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

The CUIA action is `TOGGLE_SEQ phrase,chain`. Mapping block for **Preset 1** (notes 36–51 verified):

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

If you are using a different preset, replace these note numbers with your values from Part 1.

Check the existing lines in the text area for conflicts — any note number already listed with a different action will be overridden by your new entry. Remove conflicting lines if needed.

**Verify:** 16 new lines appear in the text area with correct note numbers and TOGGLE_SEQ targets.

### Step 4 — Save and apply

Click **Save**. Zynthian applies the configuration without a full restart.

**Verify:** Page reloads with your entries still present.

### Step 5 — Test each pad

On the SMC-PAD, press each pad once. In the launcher view on the touchscreen, the corresponding slot should toggle between playing (active indicator) and stopped.

Press the same pad again — the slot should stop.

**Verify:** All 16 pads toggle their corresponding slots. No pad triggers the wrong slot.

---

## Going Further

- Add real patterns to each slot — the test patterns can be replaced with actual sequences.
- Use the SMC-PAD's PAD BANK button to switch to a second bank of 16 pads for a second scene.
- Assign the SMC-PAD's encoders to synth parameters via MIDI CC Learn.
- Save the full setup as a snapshot so the pad mappings load with a single recall.
