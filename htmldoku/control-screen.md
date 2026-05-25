# The Control Screen

The control screen is where you adjust synth engine parameters. It appears automatically after loading a preset and is the main working view during performance and sound design.

---

## Layout

```
┌──────────────────┬──────────────────┬──────────────────────────────┐
│                  │                  │  > CHAIN                     │
│   Knob 1         │   Knob 2         │    Controllers 1  ← page     │
│   Volume         │   Pan            │                              │
│   64             │    0              │  > ZynAddSubFX               │
│                  │                  │    Global        ← active    │
├──────────────────┼──────────────────┤    Oscillators               │
│                  │                  │    Filter                    │
│   Knob 3         │   Knob 4         │    Envelope                  │
│   Cutoff         │   Resonance      │    LFO                       │
│   1200 Hz        │   0.5            │    Effects                   │
│                  │                  │  > ZynAddSubFX Part2         │
└──────────────────┴──────────────────┴──────────────────────────────┘
```

**Left side (4 knobs):** show the current page's parameters. Each knob displays:
- Parameter name (abbreviated if long)
- Current value
- Arc indicator showing position within min/max range
- MIDI CC number badge (if a CC is bound)

**Right side (page list):** all available parameter pages for every processor in the chain. Sections:
- `> CHAIN` — chain-level controllers (shared across processors)
- `> EngineName` — processor-specific pages, one entry per page

The highlighted entry in the page list determines what the 4 knobs show.

---

## Modes

The control screen operates in two modes, toggled by the select button (encoder 3 or 4 push, or SW4 short):

| Mode | Encoder function | Side chain panel |
|------|----------------|-----------------|
| **Control mode** | Encoders 1–4 adjust parameter values | Hidden |
| **Select mode** | Encoder 3 scrolls page list; push changes active page | Visible |

In **select mode**, the side chain graph appears on the left. Navigate to a different processor node to switch which processor's parameters are shown. Pressing Back exits select mode without changing the active page.

---

## Page List Structure

The list on the right side is hierarchical:

```
> CHAIN
    Controllers 1      ← if chain has assigned controllers
    Controllers 2
> FluidSynth
    Reverb
    Chorus
> ZynAddSubFX
    Global
    Oscillators
    Amplitude ENV
    Filter
    Filter ENV
    LFO
    Modulation
    ...
```

**Section headers** (shown in a highlight color) are `> ProcessorName` labels — they are not selectable pages.

**Chain Controllers** section (`> CHAIN`) only appears when the chain has controllers assigned to it — for example, a chain with a send level assigned as a "favorite" control. Up to 4 controls per `Controllers N` page.

---

## Encoder Mapping

In control mode:

| Encoder | Physical position | Knob controlled |
|---------|-----------------|-----------------|
| 1 | Leftmost | Knob 1 (top-left) |
| 2 | Second from left | Knob 2 (top-right) |
| 3 | Second from right | Knob 3 (bottom-left); page scroll in select mode |
| 4 | Rightmost | Knob 4 (bottom-right) |

**Fine adjustment:** hold the encoder push-button while rotating for smaller steps. The increment size depends on the parameter's range — useful for precise tuning on parameters like frequency (Hz) or fine detuning (cents).

---

## Screen Widgets

Some engines provide a **custom widget** that replaces the generic knob display for specific parameter pages:

| Widget type | When active | Appearance |
|-------------|------------|------------|
| `envelope` | When page has envelope parameters | ADSR graphical envelope shape |
| `filter` | When page has filter parameters | Filter frequency response curve |
| `audio_file` | When a parameter points to an audio file path | File browser / waveform preview |
| Custom engine widget | Engine-specific (`setBfree`, `Aeolus`, etc.) | Drawbars, organ panel, etc. |

When a widget is active, it occupies the lower portion of the right side. The page list still shows above it, and the 4 knobs still work — they control the parameters the widget represents.

**setBfree drawbar widget:** shows 9 vertical drawbar sliders (8', 4', 2 2/3', 2', 1 3/5', 1 1/3', 1', Percussion, Rotary). Drag each bar or use the corresponding knob.

---

## MIDI CC Learning

Bind a physical controller knob, fader, or slider to any parameter:

### Step-by-Step

1. Make sure the parameter you want to bind is visible as one of the 4 knobs (navigate to the correct page).
2. **Long-press** the encoder for that knob (hold ~600ms until the knob outline turns orange/yellow).
3. The screen shows "Learning CC..." next to that knob.
4. Move the physical controller knob/fader. The first CC message received is captured.
5. The CC number appears on the knob label. Learning ends automatically.

### MIDI Learn States

| Knob border color | State |
|------------------|-------|
| Normal | Not in learn mode |
| Orange highlight | Waiting for CC (learn mode active) |
| CC badge visible | CC bound — shows CC number |

### Chain-Wide vs Per-Processor

CC bindings are stored per-processor. To clear:
- **Single processor:** processor_options → **Clean MIDI-learn**
- **Whole chain:** chain_options → **Clear MIDI Learn**

### Global MIDI Learn

Two learn modes are available:

| Mode constant | What it captures |
|---------------|-----------------|
| `MIDI_LEARNING_CHAIN` (1) | Binds CC to the selected chain's active parameter |
| `MIDI_LEARNING_GLOBAL` (2) | Binds CC to a global MIDI control |

CC bindings are saved with snapshots and with ZS3 states, so they persist across power cycles.

Source: [`zyngui/zynthian_gui_control.py:42`](../zynthian-ui/zyngui/zynthian_gui_control.py)

---

## Control XY Mode

The XY controller maps two parameters to a 2D touch pad. Drag to control both simultaneously — useful for filter + resonance, reverb wet + dry, or any two parameters you want to morph together.

### Activating XY Mode

1. On the control screen in select mode, navigate to the parameter you want as the X axis.
2. Bold-press SW4 (or the V5 equivalent) — a parameter selector opens.
3. Select the X parameter. Another selector opens for Y.
4. Select the Y parameter. The `control_xy` screen opens.

### Using the XY Pad

| Action | Effect |
|--------|--------|
| Drag on pad | Move both parameters simultaneously |
| Encoder 2 rotate | Adjust Y axis value precisely |
| Encoder 3 rotate | Adjust X axis value precisely |
| Top-left corner | Shows current X and Y parameter names and values |
| Tap screen 3× quickly | Exit XY mode (return to control screen) |
| Back button | Exit XY mode |

The crosshairs show the current position. X axis is horizontal; Y axis is vertical (top = high value, bottom = low). The value range is the full min/max range of each parameter.

**Logarithmic parameters:** if a parameter is logarithmic (e.g. filter frequency in Hz), the XY pad uses a log scale for that axis — small movements near the bottom are finer than near the top.

Source: [`zyngui/zynthian_gui_control_xy.py`](../zynthian-ui/zyngui/zynthian_gui_control_xy.py)

---

## Saving Parameter State

| Save method | Captures | Use for |
|-------------|---------|---------|
| **Snapshot** | All chains, all parameters, all routing | Between songs |
| **ZS3 (subsnapshot)** | Selected chains/parameters | Between sections within a song |
| **Preset** | Engine-internal parameters only | Reuse a sound in any setup |

To save current parameters as a ZS3: Main Menu → ZS3 → **Save as new ZS3**. The current control screen values are included.

To save as a snapshot: Main Menu → Snapshots → select a slot → Save.

---

## Parameter Types

Zynthian parameters (zctrls) come in several types. The knob display adapts:

| Type | Visual | Encoder behavior |
|------|--------|-----------------|
| Continuous float | Arc sweep 0–100% | Smooth rotation |
| Integer | Numeric display | Steps by 1 per detent |
| Enumerated | Text label | Cycles through options |
| Boolean | Toggle (on/off) | One detent = toggle |
| File path | Filename text | Opens file selector on push |
| Logarithmic | Arc with log scale | Fine at low values, coarser at high |

---

## Example: Assign Filter Cutoff to Mod Wheel (FluidSynth)

1. Load FluidSynth with any preset.
2. Control screen → navigate to the "Reverb" or "Chorus" page — find "Cutoff" (or navigate to a page that shows Cutoff).
3. Long-press encoder 3 (knob 3 = Cutoff) → knob highlights orange.
4. Move the mod wheel on your MIDI controller.
5. CC 1 is captured. Mod wheel now sweeps Cutoff.

---

## Example: Map 4 ZynAddSubFX Parameters to 4 Physical Encoders

1. Load ZynAddSubFX with a pad preset.
2. Navigate to the "Filter" page — shows Cutoff, Resonance, Filter velocity sensitivity, Filter keyboard tracking.
3. Long-press encoder 1 → move BCR2000 knob 1 → CC 21 bound to Cutoff.
4. Long-press encoder 2 → move BCR2000 knob 2 → CC 22 bound to Resonance.
5. Long-press encoder 3 → move BCR2000 knob 3 → CC 23 bound to velocity sensitivity.
6. Long-press encoder 4 → move BCR2000 knob 4 → CC 24 bound to keyboard tracking.
7. Save as ZS3 to lock this assignment to the current song section.

---

## What's Next

- [MIDI CC Learning](midi-cc-learn.html) — full CC routing and profile management
- [ZS3 Subsnapshots](zs3-guide.html) — save and recall parameter states
- [Chains & Routing](chain-management.html) — add effects to a chain
- [Performance & CPU](performance-monitoring.html) — parameter changes and CPU impact

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_control.py`, `zyngui/zynthian_gui_control_xy.py`, `zyngui/zynthian_gui_chain_control.py`.*
