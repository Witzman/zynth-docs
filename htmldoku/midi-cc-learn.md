# MIDI CC Learning & Binding

Zynthian can bind any physical controller knob, slider, or button to any synth parameter via MIDI CC. This page covers the CC learn workflow, CC routing, Program Change binding, and MIDI profiles.

---

## Overview

**CC Learn** (Continuous Controller learning) captures the next CC message your controller sends and assigns it permanently to the parameter you selected. Bindings survive snapshot saves and reloads.

**CC routing** (`midi_cc` screen) controls which of the 128 CC numbers pass through to a chain's engine. This is a filter, independent of CC learn.

**MIDI profiles** (`midi_profile` screen) are shell scripts that set global MIDI behavior — port modes, transpose, PC handling. Switching profiles reloads all MIDI configuration.

---

## MIDI CC Learn — Step-by-Step

### From the Control Screen

1. Navigate to the control screen for the chain you want to control.
2. Navigate to the page containing the parameter you want to bind (encoder 3 in select mode, or touch the page list).
3. Make sure the target parameter is one of the 4 visible knobs.
4. **Long-press the encoder** for that knob (hold ~600ms). The knob border turns orange.
5. Move the physical knob, fader, or slider on your MIDI controller.
6. The CC number is captured. The knob shows the CC number badge. Learning ends.

### Visual Feedback

| State | Knob appearance |
|-------|----------------|
| Not learning | Normal border |
| Learning active | Orange highlight border |
| CC assigned | CC number badge below knob name |
| Learn cancelled | Returns to normal |

### Cancelling Learn Without Assigning

Long-press the encoder again while in orange state, or press Back. No CC is assigned and the previous state is restored.

---

## Fine-Tuning a CC Binding

After binding, you can restrict the parameter range the CC sweeps:

1. Long-press the bound knob again (already has CC) — options menu opens.
2. Select **Set CC Range**.
3. Set **Min value** — parameter value when CC = 0.
4. Set **Max value** — parameter value when CC = 127.

**Use case:** bind mod wheel (CC 1) to filter cutoff, but restrict to 200 Hz–2000 Hz rather than the full 20 Hz–20 kHz — prevents the filter from fully closing during performance.

---

## Clearing CC Bindings

| Scope | How |
|-------|-----|
| Single parameter | Long-press the bound knob → **Clear CC** |
| All parameters in one processor | processor_options → **Clean MIDI-learn** |
| All parameters in one chain | chain_options → **Clear MIDI Learn** |

Clearing is permanent — the binding is removed from the snapshot state immediately.

---

## MIDI CC Routing Screen

Access: chain_control → MIDI config subscreen → CC routing, or via Admin → MIDI.

The `midi_cc` screen shows all 128 MIDI CC numbers for the selected chain:

```
☑ CC 00    ☑ CC 01    ☑ CC 07    ☐ CC 11
☑ CC 16    ☐ CC 17    ☑ CC 64    ☐ CC 74
...
```

- **☑ CC NN** — this CC is **routed through** to the engine
- **☐ CC NN** — this CC is **blocked** for this chain

Blocking a CC prevents it from reaching the engine regardless of CC learn. Used to prevent unwanted automation — for example, block CC 7 (volume) if your keyboard auto-sends channel volume and you want the ZynMixer to control levels instead.

**Toggle:** tap an entry or push encoder 3 while highlighted. Takes effect immediately with no restart.

Source: [`zyngui/zynthian_gui_midi_cc.py`](../zynthian-ui/zyngui/zynthian_gui_midi_cc.py)

---

## MIDI Port Configuration Screen

Access: chain_control → side chain panel → MIDI input/output node.

The `midi_config` screen lists all detected MIDI ports. Input and output are separate views (toggle via `midi_input` flag).

### Port Icons (Input Ports)

| Icon | Meaning |
|------|---------|
| `⇥` | **Active mode** — this port feeds only the active (selected) chain |
| `⇶` | **Multitimbral mode** — this port routes to all chains by MIDI channel |
| `♣` | **Sequencer capture** — this port also feeds the zynseq step sequencer |
| `⏱` | **MIDI Clock** — this port is the external clock source |
| `⌨` | **Controller driver** — a ctrldev driver is loaded for this device |

**No icon** = standard routing, multitimbral mode.

### Port States in Chain View

| Prefix | Meaning |
|--------|---------|
| ☑ (icons) port name | Connected to this chain |
| ☐ (icons) port name | Not connected to this chain |
| (indented, no ☑/☐) | Port captured by a controller driver, not available to chains |

**Toggle connection:** tap the port entry. Takes effect immediately.

**Bold-press a port:** opens options (set mode, set as clock source, assign controller driver, etc.).

Source: [`zyngui/zynthian_gui_midi_config.py`](../zynthian-ui/zyngui/zynthian_gui_midi_config.py)

---

## Program Change Binding

The `midi_prog` screen links a MIDI Program Change number to a preset.

**When it is used:** when Admin → MIDI → Program Change for ZS3 is **disabled**, PC messages load presets in the active engine. The `midi_prog` screen maps PC numbers to specific presets.

| Feature | Detail |
|---------|--------|
| Range | 0–127 plus "None" |
| Scope | Per-chain; stored with the chain state |
| Trigger | PC message on the chain's MIDI channel |

**Workflow:**

1. Load the preset you want to assign.
2. Navigate to that chain's MIDI settings (or the midi_prog screen opens automatically on PC receive).
3. Select the PC number you want to assign to this preset.
4. Next time that PC number arrives on that channel, the preset loads.

Source: [`zyngui/zynthian_gui_midi_prog.py`](../zynthian-ui/zyngui/zynthian_gui_midi_prog.py)

---

## MIDI Profiles

A MIDI profile is a `.sh` file that sets environment variables controlling MIDI behavior. Profiles live in `/zynthian/config/midi-profiles/`.

**Loading a profile:** Admin → MIDI → MIDI Profile (when exposed) — or from the `midi_profile` screen.

The screen lists all `.sh` files in the profiles directory. The currently active profile is pre-selected. Selecting a different profile calls `zynconf.save_config()` and reloads MIDI configuration immediately.

### Common Profile Variables

| Variable | Default | Effect |
|----------|---------|--------|
| `ZYNTHIAN_MIDI_FINE_TUNING` | 440 | Master tuning reference frequency (Hz) |
| `ZYNTHIAN_MIDI_TRANSPOSE` | 0 | Global MIDI note transpose (semitones) |
| `ZYNTHIAN_MIDI_PROG_CHANGE_ZS3` | 0 | 1 = PC messages recall ZS3; 0 = load presets |
| `ZYNTHIAN_MIDI_BANK_CHANGE` | 0 | 1 = process Bank Select CC 0/32 |
| `ZYNTHIAN_MIDI_FILTER_OUTPUT` | 0 | 1 = filter MIDI output |
| `ZYNTHIAN_MIDI_SINGLE_ACTIVE_CHANNEL` | 0 | 1 = active MIDI channel sends to one chain only |
| `ZYNTHIAN_MIDI_SYS_ENABLED` | 0 | 1 = process SysEx messages |
| `ZYNTHIAN_MIDI_USB_BY_PORT` | 0 | 1 = map USB devices by port number as well as name |
| `ZYNTHIAN_MIDI_CLOCK_OUTPUT_ENABLED` | 0 | 1 = send MIDI clock to output ports |

Source: [`zyngui/zynthian_gui_midi_profile.py`](../zynthian-ui/zyngui/zynthian_gui_midi_profile.py)

---

## Admin MIDI Settings

Admin → MIDI section — all global MIDI toggles and settings:

| Setting | Env var written | Detail |
|---------|----------------|--------|
| MIDI Input Devices | — | Opens midi_config in admin mode (no chain) |
| MIDI Output Devices | — | Opens midi_config output view |
| Active MIDI channel ☑/☐ | `ZYNTHIAN_MIDI_SINGLE_ACTIVE_CHANNEL` | ☑ = CC/PC only to active chain; ☐ = to all chains on that channel |
| Program Change for ZS3 ☑/☐ | `ZYNTHIAN_MIDI_PROG_CHANGE_ZS3` | ☑ = PC recalls ZS3; ☐ = PC loads presets |
| MIDI Bank Change ☑/☐ | `ZYNTHIAN_MIDI_BANK_CHANGE` | Only shown when ZS3 mode off; ☑ = Bank Select processed |
| MIDI-USB mapped by port ☑/☐ | `ZYNTHIAN_MIDI_USB_BY_PORT` | ☑ = device ID includes USB port number |
| Global Transpose (±N) | via `lib_zyncore.set_global_transpose()` | Range −24 to +24 semitones |
| Channel Pressure → CC | `ZYNTHIAN_MIDI_CHANPRESS_CC` | Map mono aftertouch to CC 0–119 |

---

## Key Ranges and Splits

The `midi_key_range` screen restricts which MIDI notes a chain receives:

| Setting | Effect |
|---------|--------|
| Min note | Lowest note this chain responds to |
| Max note | Highest note this chain responds to |

Notes outside the range are filtered before reaching the engine. Combine with two chains on the same channel to create a keyboard split (see [Chains & Routing](chain-management.html)).

---

## Example: Mapping a 16-Knob Controller (BCR2000)

The Behringer BCR2000 has 32 rotary encoders in 4 banks. Map a full bank (8 knobs) to ZynAddSubFX parameters.

**Bank 1 mapping (CC 21–28 on channel 1):**

| BCR2000 knob | CC# | ZynAddSubFX parameter |
|--------------|-----|-----------------------|
| Knob 1 | 21 | Volume |
| Knob 2 | 22 | Pan |
| Knob 3 | 23 | Filter Cutoff |
| Knob 4 | 24 | Resonance |
| Knob 5 | 25 | Amplitude Attack |
| Knob 6 | 26 | Amplitude Decay |
| Knob 7 | 27 | Amplitude Sustain |
| Knob 8 | 28 | Amplitude Release |

**Steps:**

1. Program BCR2000 to send CC 21–28 on its Bank 1 encoders, MIDI channel 1.
2. Load ZynAddSubFX on chain 1.
3. Control screen → navigate to "Global" page → long-press encoder 1 → move BCR2000 knob 1 → CC 21 bound to Volume.
4. Continue for all 8 parameters across pages.
5. Save as ZS3 "BCR2000 Bank1".

**Next session:** load the ZS3 → all 8 bindings restore → BCR2000 knobs are immediately active.

---

## Example: Saving a MIDI Profile for a Controller

Create a named profile for a Roland A-88 that transposes +12 semitones and uses ZS3 PC mode:

1. SSH into the Pi:
```bash
ssh root@zynthian.local
```

2. Copy the default profile and edit it:
```bash
cp /zynthian/config/midi-profiles/default.sh /zynthian/config/midi-profiles/roland_a88.sh
nano /zynthian/config/midi-profiles/roland_a88.sh
```

3. Set these variables:
```bash
export ZYNTHIAN_MIDI_TRANSPOSE=12
export ZYNTHIAN_MIDI_PROG_CHANGE_ZS3=1
export ZYNTHIAN_MIDI_BANK_CHANGE=0
```

4. In the Zynthian UI: Admin → (if midi_profile exposed) → select `roland_a88` → loads immediately.

---

## What's Next

- [Control Screen](control-screen.html) — where CC learn is initiated
- [ZS3 Subsnapshots](zs3-guide.html) — save CC state per song section
- [MIDI Controllers](midi.html) — connecting USB and BLE controllers
- [Admin & System](admin-guide.html) — global MIDI settings

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_midi_cc.py`, `zyngui/zynthian_gui_midi_prog.py`, `zyngui/zynthian_gui_midi_profile.py`, `zyngui/zynthian_gui_midi_config.py`, `zyngui/zynthian_gui_admin.py`.*
