# MIDI CC Learning & Binding

Zynthian can bind any physical controller knob, slider, or button to any synth parameter via MIDI CC. This page covers the CC learn workflow, CC routing screens, Program Change binding, and MIDI profiles.

---

## How MIDI CC Learn Works

CC learn captures the next CC message your controller sends and assigns it to the parameter you selected. The binding is stored per-processor and saved with snapshots.

### Step-by-Step from the Control Screen

1. Navigate to the control screen for the chain you want to control.
2. Find the parameter you want to bind (rotate to the correct page; the parameter should be visible as one of the 4 knobs).
3. **Long-press** the encoder for that knob (hold ~600ms until the knob highlights orange).
4. Move the physical knob/fader/slider on your MIDI controller.
5. The CC number is captured and shown on the knob label. The binding is active.

**Clearing a binding:** long-press the encoder again when no CC is incoming, or use chain_options → **Clear MIDI Learn** to remove all bindings from the chain at once.

---

## CC Range

After binding a CC, you can set the parameter range it controls:

1. Long-press the bound knob to open the CC range editor.
2. Set **Min** and **Max** — the parameter value range that the CC sweeps from 0 to 127.

Example: bind CC 74 (filter frequency) but limit it to the middle range (40–90 Hz) so the filter doesn't close completely.

---

## MIDI CC Screen

Access: chain_control → MIDI input subscreen → CC routing, or via admin MIDI settings.

The `midi_cc` screen shows all 128 CC numbers with checkboxes:

- ☑ CC XX — this CC is routed through to the engine
- ☐ CC XX — this CC is blocked (filtered out)

Use this screen to prevent unwanted CCs from your controller from reaching the engine. For example, if your controller sends CC 7 (volume) automatically and you don't want it to override the engine's internal volume — uncheck CC 7 on that chain.

**Toggle a CC:** tap the CC entry or push encoder 3 while it is highlighted. The change takes effect immediately — no restart needed.

Source: [`zyngui/zynthian_gui_midi_cc.py`](../zynthian-ui/zyngui/zynthian_gui_midi_cc.py)

---

## Program Change Binding

The `midi_prog` screen lets you assign a MIDI Program Change number to a preset:

- Access: usually opened automatically when the engine receives a PC message and `Program Change for ZS3` is off.
- Shows 0–127 numbered slots plus a "None" option.
- Select a slot to link the current preset to that PC number.

After binding, when your controller sends PC#N on the chain's MIDI channel, that preset loads automatically.

**Note:** if "Program Change for ZS3" is enabled in Admin → MIDI, PC messages recall ZS3 states instead of individual presets. The `midi_prog` screen is only for preset binding when ZS3 mode is off.

Source: [`zyngui/zynthian_gui_midi_prog.py`](../zynthian-ui/zyngui/zynthian_gui_midi_prog.py)

---

## MIDI Profiles

A MIDI profile is a `.sh` environment file that sets MIDI routing and CC mapping for a specific controller. Profiles live in `/zynthian/config/midi-profiles/`.

**Loading a profile:**

1. Admin → MIDI → MIDI Profile (or chain_options → MIDI profile if accessible).
2. The `midi_profile` screen lists all `.sh` files in the profiles directory.
3. Select a profile — it loads immediately and reloads the MIDI configuration.

**What a profile contains:**

```bash
ZYNTHIAN_MIDI_FINE_TUNING=440
ZYNTHIAN_MIDI_TRANSPOSE=0
ZYNTHIAN_MIDI_PROG_CHANGE_ZS3=0
ZYNTHIAN_MIDI_FILTER_OUTPUT=0
```

The `default.sh` profile is loaded at boot. Create named profiles for specific controllers — e.g. `bcr2000.sh` for a Behringer BCR2000.

Source: [`zyngui/zynthian_gui_midi_profile.py`](../zynthian-ui/zyngui/zynthian_gui_midi_profile.py)

---

## MIDI Key Range

The `midi_key_range` screen restricts which MIDI notes a chain responds to. Set Min and Max notes to define the playable range. Combined with multiple chains, this creates keyboard splits.

Access via chain_options → MIDI channel settings, or through the chain_control MIDI subscreen.

---

## Example: Mapping a 16-Knob Controller (Behringer BCR2000)

Goal: map 16 physical knobs to ZynAddSubFX parameters across 4 pages.

**Setup:**

1. Load ZynAddSubFX with any preset.
2. Control screen → navigate to "Part 1" page.
3. Long-press encoder 1 (Volume knob) → move BCR2000 knob 1 → CC 16 bound.
4. Long-press encoder 2 (Pan knob) → move BCR2000 knob 2 → CC 17 bound.
5. Long-press encoder 3 (Cutoff knob) → move BCR2000 knob 3 → CC 18 bound.
6. Long-press encoder 4 (Resonance knob) → move BCR2000 knob 4 → CC 19 bound.
7. Navigate to next page → bind knobs 5–8 → CCs 20–23.
8. Continue for remaining pages.

**Save the controller mapping:**

After binding all knobs, save as a ZS3 to capture the parameter assignments. The CC bindings themselves are saved with the snapshot.

---

## Example: Saving a MIDI Profile for a Specific Controller

When you have a controller with fixed CC assignments and want to recall them on every boot:

1. Set up MIDI settings in Admin → MIDI (channels, active MIDI, transpose, etc.).
2. The settings are written to `zynthian_envars.sh`.
3. To make a named profile: copy `/zynthian/config/zynthian_envars.sh` to `/zynthian/config/midi-profiles/mycontroller.sh` (via SSH).
4. Load the profile via the midi_profile screen — it becomes active immediately and can be re-selected on future sessions.

---

## What's Next

- [Control Screen](control-screen.html) — where CC learn is initiated
- [ZS3 Subsnapshots](zs3-guide.html) — save CC state per song section
- [MIDI Controllers](midi.html) — connecting USB and BLE controllers

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_midi_cc.py`, `zyngui/zynthian_gui_midi_prog.py`, `zyngui/zynthian_gui_midi_profile.py`.*
