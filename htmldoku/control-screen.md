# The Control Screen

The control screen is where you adjust synth engine parameters. It appears automatically after loading a preset and is the main view during performance.

---

## Layout

```
┌──────────┬──────────┬──────────────────────────┐
│          │          │  > Global                │
│ Knob 1   │ Knob 2   │    Volume        ████    │
│ Volume   │ Pan      │    Pan           ████    │
│          │          │  > Filter                │
├──────────┼──────────┤    Cutoff        ████    │
│          │          │    Resonance     ████    │
│ Knob 3   │ Knob 4   │  > Envelope              │
│ Cutoff   │ Resonance│    Attack        ████    │
│          │          │    Release       ████    │
└──────────┴──────────┴──────────────────────────┘
```

**Left side:** 4 parameter knobs. Each knob shows:
- Parameter name
- Current value (text and arc indicator)
- MIDI CC assignment (if learned)

**Right side:** List of all parameter pages for the current processor. Highlighted page = what the 4 knobs are showing. Scroll this list to switch pages.

---

## Modes

The control screen has two modes, toggled by the select button (SW4):

| Mode | Encoders | What happens |
|------|---------|-------------|
| **Control mode** | Adjust parameter values | Encoder 1–4 move the 4 visible knobs |
| **Select mode** | Choose active processor | Encoder 3 scrolls the page list; push selects |

In **select mode**, the side chain graph becomes visible (if in chain_control). Navigate to a different processor node to switch which processor's parameters are shown.

---

## Encoder Mapping

In control mode, each encoder controls one on-screen knob:

| Encoder | Knob |
|---------|------|
| 1 (leftmost) | Knob 1 (top-left) |
| 2 | Knob 2 (top-right) |
| 3 | Knob 3 (bottom-left) — also scrolls page list in select mode |
| 4 | Knob 4 (bottom-right) |

**Fine adjustment:** hold the encoder button while rotating for small incremental changes.

---

## Parameter Pages

Each engine exposes its parameters in groups called **pages**. A page holds up to 4 parameters — one per knob. To switch pages:

- **Encoder 3 in select mode:** scroll the right-hand list and push to select a page.
- **Touch:** tap any page name in the right-side list.

The number of pages depends on the engine. ZynAddSubFX has dozens; setBfree has two (drawbars, rotary).

---

## MIDI CC Learning

Bind a physical controller knob/fader to any parameter:

1. Navigate to the parameter you want to bind (it should be on one of the 4 visible knobs).
2. **Long-press** that knob's encoder button (or touch the knob widget and long-press).
3. The knob highlights in orange — it's now waiting for a CC message.
4. Move the physical controller knob/slider you want to use.
5. The CC number is captured and the binding is saved.

To **clear** a CC binding: long-press the same knob again when no CC is coming in, or use chain_options → **Clear MIDI Learn** to remove all bindings from the chain.

CC bindings are saved with snapshots and with ZS3 states.

Source: [`zyngui/zynthian_gui_control.py:42`](../zynthian-ui/zyngui/zynthian_gui_control.py)

---

## Control XY Mode

The XY controller maps two parameters to a 2D touch pad — drag to control both simultaneously.

**Activating XY mode:**

1. On the control screen, select a parameter for the X axis: long-press SW4 (bold select).
2. A second selection prompt appears for the Y axis.
3. The control_xy screen opens.

**In control_xy:**
- Drag finger across the pad to move both parameters.
- Crosshairs show current position.
- Current X/Y values shown in top-left corner.
- Encoders 3 and 4 also work: encoder 2 = Y axis, encoder 3 = X axis.
- **Exit:** tap the screen 3 times quickly, or press Back.

Source: [`zyngui/zynthian_gui_control_xy.py`](../zynthian-ui/zyngui/zynthian_gui_control_xy.py)

---

## Widgets

Some engines provide a **custom widget** instead of generic knobs. Examples:

- **setBfree:** drawbar widget — vertical fader bars for each drawbar
- **ZynAddSubFX:** standard knobs
- **Jalv LV2:** knobs mapped to whatever the plugin exposes

If a widget is active, the standard knobs are hidden and the widget fills the right half of the screen.

---

## Saving Parameter State

Control screen parameter values are saved in two ways:

| Method | When to use |
|--------|-------------|
| **Snapshot** | Full save — all chains, all parameters, all routing |
| **ZS3** | Partial save — specific parameters per chain; recalled by Program Change |

To save current state: Main Menu → **Snapshots** → select slot → Save. Or use ZS3 screen for a quick sub-snapshot.

---

## Example: Assign Filter Cutoff to Mod Wheel (FluidSynth)

1. Load FluidSynth with any preset.
2. Navigate to the "Filter" page in the control screen.
3. Knob 3 shows "Cutoff" — long-press encoder 3 to enter learn mode.
4. Move the mod wheel on your MIDI controller.
5. CC 1 (mod wheel) is now bound to Cutoff.

---

## Example: Map 4 ZynAddSubFX Parameters to Encoders

1. Load ZynAddSubFX with any preset.
2. Go to the "Part Effects" page — shows Reverb Send, Chorus Send, Bandwidth, Modulation.
3. Long-press each encoder in sequence and move a physical knob for each.
4. Now 4 physical knobs directly control those 4 synth parameters.
5. Save as a ZS3 to recall this mapping quickly.

---

## What's Next

- [MIDI CC Learning](midi-cc-learn.html) — full CC routing and profile management
- [ZS3 Subsnapshots](zs3-guide.html) — save and recall parameter states
- [Chains & Routing](chain-management.html) — add effects to a chain

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_control.py`, `zyngui/zynthian_gui_control_xy.py`, `zyngui/zynthian_gui_chain_control.py`.*
