# MIDI Reference

**Purpose:** Complete MIDI map for this hardware rig — device capabilities, active assignments, conflicts, and Zynthian feature triggers. Update this page whenever a tutorial is verified or configuration changes deliberately.

**Hardware:** Maschine MK2 (via MaschineMK2_linux daemon) · E-MU Xboard 25 · SMC-PAD (Preset 1)
**Access:** SSH · Webconf

---

## Section 1 — Device Capabilities

### Maschine MK2 (MaschineMK2_linux daemon)

The Maschine MK2's native USB MIDI port sends no pad data without NI software. All pad MIDI comes from the `MaschineMK2_linux` daemon, which reads HID and outputs ALSA MIDI via the `Pads MIDI` port on **channel 1** (normal mode) and **channel 2** (sequencer mode).

| Control | MIDI message | Ch | Range / values |
|---|---|---|---|
| 16 pads — normal mode | Note On / Note Off | 1 | base + offset (see layout below), velocity from pressure |
| 16 pads — sequencer mode | Note On / Note Off | 2 | stored per step |
| 8 encoders | **RPN14** | 1 | RPN numbers 16–23, values 0–8191 |
| ~30 transport / function buttons | **RPN7** | 1 | RPN numbers 1–48 (see table below) |
| Group buttons A–H | *(none — internal state only)* | — | sets note base: A=24 B=36 C=48 D=60 E=72 F=84 G=96 H=108 |

> **Zynthian limitation:** Encoders send RPN14, transport buttons send RPN7. Neither is standard CC 0–119. Zynthian CC Learn cannot capture them. MIDI filter rules remapping RPN → CC are required before any binding is possible.

**Pad note layout** — offsets added to current note base. Pad rows top → bottom, left → right:

```
Row 0 (top): +12  +13  +14  +15
Row 1:        +8   +9  +10  +11
Row 2:        +4   +5   +6   +7
Row 3 (bot):  +0   +1   +2   +3
```

With Group D active (base 60 = C4):

```
Row 0: C4  C#4  D4  D#4   (MIDI 60–63)
Row 1: G#3  A3  A#3  B3   (MIDI 56–59)
Row 2: E3   F3  F#3  G3   (MIDI 52–55)
Row 3: C3  C#3  D3  D#3   (MIDI 48–51)
```

**Transport / function button RPN7 map (Ch 1):**

| Button | RPN | Button | RPN |
|---|---|---|---|
| Play | 1 | Main | 24 |
| Stop (Erase) | 2 | Scene | 25 |
| Rec | 3 | Pattern | 26 |
| Grid | 4 | Pad Mode | 27 |
| Step Left | 5 | View | 28 |
| Step Right | 6 | Duplicate | 29 |
| Restart | 7 | Select | 30 |
| Browse | 8 | Solo | 31 |
| Sampling | 9 | Step | 32 |
| Note Repeat | 10 | Mute | 33 |
| Control | 11 | Navigate | 34 |
| Nav | 12 | Tempo | 35 |
| Nav Left | 13 | Enter | 36 |
| Nav Right | 14 | Auto | 37 |
| — | — | All | 38 |
| — | — | F1–F8 | 39–46 |
| — | — | Page Right | 47 |
| — | — | Page Left | 48 |

**OSC interface:** daemon listens on `127.0.0.1:42434`, sends to `42435`. Supports remote LED control (`/maschine/button/<name>`, `/maschine/pad`) and setting MIDI note base (`/maschine/midi_note_base`).

---

### E-MU Xboard 25

| Control | MIDI message | Ch | Range | Notes |
|---|---|---|---|---|
| 25 keys | Note On / Note Off | patch ch | full range ± 4 oct transpose | velocity-sensitive |
| Channel aftertouch | Channel Pressure | patch ch | 0–127 | configurable on/off |
| Pitch wheel | Pitch Bend | patch ch | ±8192 | springs to center |
| Mod wheel | CC 1 | patch ch | 0–127 | CC number reassignable |
| 16 CC knobs | CC | per-knob ch | 0–127 | **any CC 0–127, any channel, per patch** |
| Footswitch | CC 64 (sustain) | patch ch | 0 / 127 | or continuous pedal mode |
| Data slider | Universal Real Time SysEx | all | master volume | not standard CC |
| Snapshot button | bulk CC send | — | stored knob positions | sends all 16 knobs + wheels at once |
| Program Change | PC + Bank Select (CC 0 + CC 32) | per ch | 0–127 | per patch, per channel |
| Panic (Oct↑ + Oct↓ simultaneously) | All Notes Off + All Sounds Off + CC 64=0 | ch 1–16 | — | clears stuck notes |
| 4 Zones | keys split across 4 MIDI channels | 4 ch | key + velocity range per zone | requires Xboard Control software |

> **[low] Factory CC defaults for the 16 knobs are not documented in the text manual.** Verify with `amidi -d -p hw:X,0,0` (replace X with Xboard card number from `aconnect -l`) before assigning any knob. Until verified, treat CC 1, 7, 10, 11, 16–19, 30, 64, 71, 74 as potentially occupied.

---

### SMC-PAD (Preset 1 — Performance preset)

Select Preset 1: **Shift + Pad 1**.

All data below verified from tutorial testing, ctrldev driver source code, and live MIDI capture on Pi (2026-06-05). **Pad channel is 6, not 7** — confirmed from raw MIDI debug (`status 0x95` = Note-On ch6 1-indexed).

| Control | MIDI message | Ch | Value |
|---|---|---|---|
| Pad 1 (bottom-left) | Note 36 On/Off | **6** | velocity-sensitive + aftertouch |
| Pad 2 | Note 37 | **6** | — |
| Pad 3 | Note 38 | **6** | — |
| Pad 4 | Note 39 | **6** | — |
| Pad 5 | Note 40 | **6** | — |
| Pad 6 | Note 41 | **6** | — |
| Pad 7 | Note 42 | **6** | — |
| Pad 8 | Note 43 | **6** | — |
| Pad 9 | Note 44 | **6** | — |
| Pad 10 | Note 45 | **6** | — |
| Pad 11 | Note 46 | **6** | — |
| Pad 12 | Note 47 | **6** | — |
| Pad 13 | Note 48 | **6** | — |
| Pad 14 | Note 49 | **6** | — |
| Pad 15 | Note 50 | **6** | — |
| Pad 16 (top-right) | Note 51 | **6** | — |
| Knob 7 (top-left col) | CC 16 | ? | 0–127 absolute |
| Knob 5 | CC 17 | ? | 0–127 absolute |
| Knob 3 | CC 18 | ? | 0–127 absolute |
| Knob 1 (bottom-left col) | CC 30 | ? | 0–127 absolute |
| Knob 8 (top-right col) | CC 80 | ? | 0–127 absolute |
| Knob 6 | CC 81 | ? | 0–127 absolute |
| Knob 4 | CC 82 | ? | 0–127 absolute |
| Knob 2 (bottom-right col) | CC 31 | ? | 0–127 absolute |
| Transport Left | CC 25 | 1 | 127 on press, 0 on release |
| Transport Right | CC 26 | 1 | 127 on press, 0 on release |

> **[low] Encoder MIDI channel (marked ?) — verify with `amidi -d` capture.**

**ctrldev driver (`zynthian_ctrldev_sinco_smc_pad.py`) — active when SINCO IN 2 port has driver assigned:**

| Incoming | Action | Notes |
|---|---|---|
| CC 16 → | ZYNPOT_ABS 0 (top screen knob) | absolute position |
| CC 17 → | ZYNPOT_ABS 1 | — |
| CC 18 → | ZYNPOT_ABS 2 | — |
| CC 30 → | ZYNPOT_ABS 3 (bottom screen knob) | — |
| CC 25 (press=127) → | PROGRAM_CHANGE − on drum chain | drum chain = MIDI ch 7, hardcoded |
| CC 26 (press=127) → | PROGRAM_CHANGE + on drum chain | drum chain = MIDI ch 7, hardcoded |
| Pad notes 36–51 ch **6** | pass through to chains | TOGGLE_SEQ still fires |

PAD BANK and KNOB BANK buttons switch to bank 2 assignments — not yet mapped.
Preset 2 (DAW): **Shift + Pad 2** — transport buttons send Mackie Control; use in DAW mode only.

---

## Section 2 — Master MIDI Assignment Matrix

### Global settings (live — `default.sh`)

| Setting | Value | Effect |
|---|---|---|
| Master MIDI channel | **6** | Notes on ch 6 trigger CUIA master key actions |
| Master key actions | notes 36–51 → `TOGGLE_SEQ 0,0–3,3` | All 16 configured — see Conflict 10 for why toggle appears broken |
| MIDI filter rules | *none* | No RPN→CC remapping, no channel redirects |
| SINGLE_ACTIVE_CHANNEL | **ON** | All MIDI routes to the currently selected chain |
| PROG_CHANGE_ZS3 | **ON** | Program Change recalls ZS3 subsnapshots |
| BLE MIDI | enabled | SMC-PAD BLE broken on kernel 6.12 — use USB |

### JACK device routing (live)

| ZynMidiRouter port | Physical device |
|---|---|
| dev0_in | Maschine MK2 native USB MIDI *(no pad data — needs daemon)* |
| dev1_in | E-MU Xboard 25 |
| dev2_in | SINCO SMC-PAD Private port |
| dev3_in | SINCO SMC-PAD Master port (pads + CCs) **and** Maschine daemon `Pads MIDI` (fan-in) |
| dev4_in | SINCO SMC-PAD port 2 |
| dev5_in | ttymidi (DIN-5) |

### Assignment matrix

Status tags: `[verified]` = Pi-tested · `[draft]` = written, not yet tested · `[low]` = not yet verified · `[blocked]` = depends on unresolved issue · `passive` = standard MIDI, no explicit assignment needed

| Device | Control | Ch | Message | Zynthian target | Tutorial | Status |
|---|---|---|---|---|---|---|
| Maschine MK2 | Pads — Group C (default) | 1 | Note 48–63 | active chain | Maschine MK2 P1 | `[verified]` |
| Maschine MK2 | Pads — Group D | 1 | Note 60–75 | active chain | Step Seq P2 | `[draft]` |
| Maschine MK2 | Sequencer output | 2 | Note (any) | chain on ch 2 | Step Seq P1 | `[draft]` |
| Maschine MK2 | 8 Encoders | 1 | RPN14 16–23 | **unmapped** | Maschine MK2 P2 | `[blocked]` |
| Maschine MK2 | Transport buttons | 1 | RPN7 1–48 | **unmapped** | Maschine MK2 P2 | `[blocked]` |
| Xboard 25 | Keys | 1 | Note 0–127 | active chain | MIDI Channel Routing | `[draft]` |
| Xboard 25 | 16 CC knobs | 1 | CC [unknown] | **unassigned** | Xboard CC Knob Map | `[draft]` |
| Xboard 25 | Mod wheel | 1 | CC 1 | active chain engine | — | passive |
| Xboard 25 | Pitch wheel | 1 | Pitch Bend | active chain engine | — | passive |
| Xboard 25 | Aftertouch | 1 | Channel Pressure | active chain engine | — | passive |
| SMC-PAD | Pads 1–12 | **6** | Note 36–47 | **unassigned** | SMC-PAD Launcher P3 | `[draft]` |
| SMC-PAD | Pad 13 (note 48) | **6** | Note 48 | TOGGLE_SEQ 0,0 | SMC-PAD Launcher P3 | partial `[low]` |
| SMC-PAD | Pads 14–16 (notes 49–51) | **6** | Note 49–51 | **unassigned** | SMC-PAD Launcher P3 | `[draft]` |
| SMC-PAD | Encoders left col | ? | CC 16/17/18/30 | Screen knobs 1–4 (ZYNPOT_ABS) | SMC-PAD Launcher P4 | `[verified]` |
| SMC-PAD | Encoders right col | ? | CC 80/81/82/31 | **unassigned** | — | — |
| SMC-PAD | Transport Left | 1 | CC 25 | PROGRAM_CHANGE − (drum ch 7) | SMC-PAD Drum Computer | `[low]` |
| SMC-PAD | Transport Right | 1 | CC 26 | PROGRAM_CHANGE + (drum ch 7) | SMC-PAD Drum Computer | `[low]` |

### Currently loaded chains

From `dub-techno-p1` snapshot (bank 000):

| Chain | MIDI ch | Engine |
|---|---|---|
| 1 | 1 | FluidSynth — drums |
| 2 | 2 | ZynAddSubFX — bass |

---

## Section 3 — Conflicts and Design Decisions

### Conflict 1 — Maschine + Xboard both on MIDI ch 1

Both devices send notes on ch 1. With SINGLE_ACTIVE_CHANNEL=ON, both drive whichever chain is currently selected. Can be intentional layering or accidental doubling.

**Resolution:** Set Xboard to ch 3 or ch 4 when used alongside Maschine. Document channel assignment in each tutorial that uses both simultaneously. *(Not yet implemented — pending MIDI Channel Routing tutorial.)*

---

### Conflict 2 — Maschine note range overlaps SMC-PAD pads

| Group | Note base | Notes sent | Overlap with SMC-PAD (36–51) |
|---|---|---|---|
| A | 24 | 24–39 | partial — notes 36–39 (pads 1–4) |
| B | 36 | 36–51 | **full overlap** |
| C | 48 | 48–63 | partial — notes 48–51 (pads 13–16) |
| D | 60 | 60–75 | none |
| E+ | 72+ | 72–87+ | none |

**Resolution:** Use Group D or higher when SMC-PAD launcher is active. Note in each affected tutorial.

---

### Conflict 3 — Master channel 6 = SMC-PAD channel

Any device sending notes on ch 6 fires CUIA master key actions. Currently only note 48 is mapped, but if all 16 mappings are added, notes 36–51 on ch 6 will all fire TOGGLE_SEQ.

**Resolution:** Ch 6 is reserved for SMC-PAD exclusively. Xboard must never be set to ch 6. Maschine Group B (note base 36) must not be used when SMC-PAD is connected.

---

### Conflict 4 — Only 1 of 16 TOGGLE_SEQ mappings active

`default.sh` MASTER_NOTE_CUIA contains only `48: TOGGLE_SEQ 0,0`. SMC-PAD Launcher P3 planned:

```
36: TOGGLE_SEQ 0,0   37: TOGGLE_SEQ 0,1   38: TOGGLE_SEQ 0,2   39: TOGGLE_SEQ 0,3
40: TOGGLE_SEQ 1,0   41: TOGGLE_SEQ 1,1   42: TOGGLE_SEQ 1,2   43: TOGGLE_SEQ 1,3
44: TOGGLE_SEQ 2,0   45: TOGGLE_SEQ 2,1   46: TOGGLE_SEQ 2,2   47: TOGGLE_SEQ 2,3
48: TOGGLE_SEQ 3,0   49: TOGGLE_SEQ 3,1   50: TOGGLE_SEQ 3,2   51: TOGGLE_SEQ 3,3
```

**Resolution:** Complete SMC-PAD Launcher P3. Add all 16 lines in webconf → MIDI Options → Master Key Actions.

---

### Conflict 5 — Maschine RPN14/RPN7 invisible to CC Learn

Encoders (RPN14 16–23) and transport buttons (RPN7 1–48) cannot be captured by CC Learn. Blocks all Maschine MK2 Part 2 work.

**Resolution:** Design MIDI filter rules (`ZYNTHIAN_MIDI_FILTER_RULES`) to remap selected RPNs to CC numbers before the signal reaches chains. Example: `RPN 16 CH1 => CC 20 CH1`. Requires Maschine MK2 P2 redesign.

---

### Conflict 6 — Xboard CC defaults unknown

Factory CC numbers for the 16 knobs not documented in text manual. Risk of collision with SMC-PAD CCs (16/17/18/30/80/81/82/31) or standard engine parameters.

**Resolution:** Before assigning any Xboard knob, run:

```bash
amidi -d -p hw:X,0,0    # replace X with Xboard card number from aconnect -l
```

Turn each knob and record CC number. Map conflicts before any tutorial uses the knobs.

---

### Conflict 7 — ctrldev DRUM_CHAN hardcoded to ch 7

`zynthian_ctrldev_sinco_smc_pad.py` has `DRUM_CHAN = 6` (0-indexed = ch 7). SMC-PAD transport Left/Right cycle drum kits only if drum chain is on ch 7. Dub Techno snapshot puts drums on ch 1 — transport buttons target wrong chain if loaded with driver active.

**Resolution:** Maintain separate snapshots per use case: SMC-PAD Drum Computer snapshot = drums on ch 7; Dub Techno snapshot = drums on ch 1 (transport buttons inactive in this context). Or update driver `DRUM_CHAN` constant when switching snapshots.

---

### Conflict 8 — CC 25/26 ch 1 reserved for SMC-PAD transport

If any Xboard knob maps to CC 25 or CC 26 on ch 1, it unintentionally cycles drum kits.

**Resolution:** Reserve CC 25 and CC 26 on ch 1 for SMC-PAD transport buttons only.

---

### Conflict 9 — dev3_in fan-in: SMC-PAD Master + Maschine daemon

Both SINCO SMC-PAD Master and the Maschine daemon `Pads MIDI` port connect to `ZynMidiRouter:dev3_in` (JACK allows multiple inputs to one port). Both are active simultaneously. No hard conflict as long as channel discipline holds (Maschine on ch 1, SMC-PAD on ch 6). If USB enumeration order changes, SMC-PAD may shift off `system:midi_capture_4` and `dev3_in` would wire Maschine to a different slot.

**Resolution:** Monitor in testing. If enumeration shifts, run `aconnect -l` and `jack_lsp -c` to diagnose.

---

### Conflict 10 — SINCO Private port mirrors all pad MIDI (double CUIA firing)

SINCO SMC-PAD has three ALSA ports. Port 0 (Private = SINCO IN 1 = `system:midi_capture_3`) is supposed to carry internal device messages only — but on this firmware it mirrors all pad notes from Port 1 (Master = SINCO IN 2 = `system:midi_capture_4`). Both ports are connected to ZynMidiRouter by autoconnect. Both fire as master channel events, causing TOGGLE_SEQ (and all other master-channel CUIAs) to fire **twice per pad press** — double-toggle = no net change.

**Confirmed:** live MIDI debug showed `EV izmip=2 head=0x95 chan=5` and `EV izmip=3 head=0x95 chan=5` for a single pad press — identical events from both ports.

**Workaround (applied):** 50ms debounce on `cuia_queue.put_nowait` in `zynthian_state_manager.py` — same note within 50ms fires CUIA only once. Change is on the Pi, not committed to the zynthian-ui git repo.

**Permanent fix options:**
1. Patch `zynautoconnect` to skip ports whose JACK alias starts with `USB:…/SINCO IN 1`
2. Use `lib_zyncore.zmip_set_flags(izmip, flags & ~FLAG_ZMIP_UI)` after identifying the Private port's zmip index at startup — clears UI flag so master-channel events from that port are silently consumed and not sent to Python

**`ZYNTHIAN_MIDI_PORTS DISABLED_IN` does NOT work** — field exists in config but is not enforced by current autoconnect code.

---

## Section 4 — Zynthian MIDI Feature Map

What Zynthian can receive and respond to. Check here before assigning a device control to avoid collisions.

| Feature | MIDI trigger | Ch scope | Configure via |
|---|---|---|---|
| Play note on chain | Note On | per-chain MIDI ch | Chain Options → MIDI Channel |
| Route all to active chain | Note On (any ch) | any | SINGLE_ACTIVE_CHANNEL=ON (currently ON) |
| CC → synth engine parameter | CC 0–119 | per-chain ch | CC Learn: long-press param knob ~600 ms |
| CC → screen knob (absolute) | CC 16 / 17 / 18 / 30 | any | ctrldev driver active on SINCO IN 2 |
| Volume | CC 7 | any | always active |
| Sustain | CC 64 | any | always active |
| Modulation | CC 1 | any | always active |
| Preset recall | Program Change | active chain ch | standard |
| ZS3 subsnapshot recall | Program Change | active chain ch | PROG_CHANGE_ZS3=ON *(currently ON)* |
| Bank select | CC 0 (MSB) + CC 32 (LSB) + PC | active chain ch | MIDI_BANK_CHANGE setting |
| Launcher slot toggle | Note On on master ch (6) | ch 6 only | webconf → MIDI Options → Master Key Actions |
| Any CUIA action | Note On on master ch (6) | ch 6 only | webconf → MIDI Options → Master Key Actions |
| Drum kit cycle (via ctrldev) | CC 25 / CC 26 press on ch 1 | ch 1 only | ctrldev driver active |
| Channel aftertouch | Channel Pressure | per-chain ch | if engine supports |
| Poly aftertouch | Poly Pressure | per-chain ch | if engine supports |
| Pitch bend | Pitch Bend | per-chain ch | standard |
| Panic | CC 123 (All Notes Off) | any | standard |
| Pitch bend range | RPN 0 | standard | standard |
| Fine tuning | RPN 1 / RPN 2 | standard | standard |
| RPN14 / RPN7 (Maschine encoders/buttons) | — | — | **not natively supported** — needs MIDI filter RPN→CC rule |

---

## Going Further

- Complete SMC-PAD Launcher P3: add all 16 TOGGLE_SEQ mappings in webconf
- Verify Xboard CC defaults via `amidi -d`, then design Xboard CC Knob Mapping tutorial
- Design MIDI filter rules for Maschine RPN→CC remapping, enabling Maschine MK2 P2
- Verify SMC-PAD encoder MIDI channel with `amidi -d`
- Map SMC-PAD bank 2 (PAD BANK / KNOB BANK assignments)
- Update this page after each tutorial is verified — change `[draft]` → `[verified]` in the assignment matrix
