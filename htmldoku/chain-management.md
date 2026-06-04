# Chains & Routing

A **chain** is the core unit of Zynthian: one MIDI input → one or more processors → one audio output. This page covers how to create, configure, and route chains, including the chain manager, chain options, processor options, and audio routing.

---

## Chain Types

When you add a chain, you choose its type. The type determines what signal flows through it and which engines are available.

| Type | MIDI in | Audio in | Audio out | MIDI thru | Use for |
|------|---------|----------|-----------|-----------|---------|
| **Instrument** | Yes | No | Yes | No | Synth engines: FluidSynth, ZynAddSubFX, setBfree |
| **Audio Input** | No | From interface | Yes | No | Live audio with effects (mic, guitar) |
| **Clip Launcher** | No | No | Yes | No | Audio clip playback; max 16 instances |
| **MIDI** | Yes | No | No | Yes | MIDI processing tools (arpeggiator, transpose) |
| **MIDI + Audio** | Yes | Yes | Yes | Yes | Combined MIDI/audio processor |
| **Audio Generator** | No | No | Yes | No | Tone generators, file playback |
| **Special** | Yes | Yes | Yes | Yes | Catch-all for special engines (MOD-UI, SooperLooper) |
| **Mixbus** | No | Yes | Yes | No | Group bus / parallel send effects |

The chain type maps internally to `type`, `midi_thru`, and `audio_thru` flags on the chain object. Source: [`zyngui/zynthian_gui_add_chain.py:42`](../zynthian-ui/zyngui/zynthian_gui_add_chain.py)

> **Clip Launcher** is limited to 16 instances per session. The button grays out and shows "Max. 16 reached!" if the limit is hit.

---

## Adding a Chain

**From the mixer:** tap the **+** strip at the right edge.

**From chain_manager:** tap the **Add chain** button.

The full flow:

```
add_chain (pick type)
    ↓
engine category list
    ↓
engine list
    ↓
bank list
    ↓
preset list
    ↓
control screen (ready to play)
```

Each step filters what follows. Selecting "Instrument" → "Sound Synthesis" shows only synth engines; selecting "LV2 Plugin" shows only LV2 plugins.

---

## Chain Manager

**From the touch keypad:** Tap **OPT/ADMIN** (short) → **Chain Manager**.

**V5 hardware:** SW1 bold from chain_control.

The chain manager shows all chains as columns in a visual graph. Each column:
- Column header: chain name + MIDI channel
- Processor nodes stacked vertically: MIDI-FX → synth/engine → Audio-FX
- Lines connecting serial and parallel processors

### Chain Manager Interactions

| Action | Result |
|--------|--------|
| Tap column header | Select that chain; enter chain_control for it |
| Tap a processor node | Enter chain_control focused on that processor |
| Long-press column header | Open chain_options for that chain |
| Long-press a processor node | Open processor_options for that processor |
| Encoder 3 rotate | Scroll between chains |
| Encoder 4 rotate | Scroll between processor nodes |
| Bold SW4 (V5 hardware) or tap **+** in Mixer | Add chain |

### Moving a Chain

chain_options → **Move chain** — the chain manager opens with the chain highlighted. Use encoder 4 to slide the chain left/right to a new position. Confirm with select.

Source: [`zyngui/zynthian_gui_chain_manager.py`](../zynthian-ui/zyngui/zynthian_gui_chain_manager.py)

---

## Chain Control

The `chain_control` screen is a compound view: a **side chain graph** panel on the left plus a **subscreen** on the right. The two parts interact:

```
┌──────────────────────────────────────────────────┐
│  [Side chain graph]  │  [Active subscreen]       │
│                      │                           │
│  MIDI-FX node        │  control / chain_options  │
│    ↓                 │  / midi_config            │
│  Synth engine node   │  / audio_out / audio_in   │
│    ↓                 │                           │
│  Audio-FX node       │                           │
└──────────────────────────────────────────────────┘
```

### Subscreens

| Subscreen | Content |
|-----------|---------|
| `control` | Parameter knobs for the selected processor |
| `chain_options` | Chain-level management options |
| `midi_config` | MIDI input or output port routing |
| `audio_out` | JACK audio output assignments |
| `audio_in` | JACK audio input assignments |

**Toggle side chain panel:** SW3 short press, or tap the left edge of the screen.

**When side chain is visible:** encoder 2 navigates between processor nodes in the graph. Selecting a node switches the right subscreen to that processor's content.

Source: [`zyngui/zynthian_gui_chain_control.py`](../zynthian-ui/zyngui/zynthian_gui_chain_control.py)

---

## Chain Options

Long-press a chain header in chain_manager, or from chain_control when the chain_options subscreen is active.

| Option | Detail |
|--------|--------|
| **Clear MIDI Learn** | Remove all CC bindings from all parameters in all processors in this chain |
| **Rename chain** | Opens keyboard screen; clear name to reset to default |
| **Move chain** | Reposition in mixer — encoder 4 slides left/right |
| **Add MIDI-FX processor** | Append a MIDI effect at the end of the MIDI chain |
| **Add Audio-FX processor** | Append an audio effect at the end of the audio chain |
| **Remove all MIDI-FX** | Strip all MIDI effect processors (with confirmation) |
| **Remove all Audio-FX** | Strip all audio effect processors (with confirmation) |
| **Remove chain** | Delete chain and all its processors (with confirmation) |
| **Export chain as snapshot** | Save this chain as a `.zss` file to a selected folder |
| **Insert new chain** | Create a new chain immediately before this one |
| **Remove ALL** | Sub-menu: remove all chains / remove all sequences / remove all chains & sequences |

The "Remove ALL" sub-menu uses a grid selector with confirmation options. Choose carefully.

Source: [`zyngui/zynthian_gui_chain_options.py`](../zynthian-ui/zyngui/zynthian_gui_chain_options.py)

---

## Processor Options

Long-press any **processor node** in the chain graph.

| Option | When shown | Detail |
|--------|-----------|--------|
| **Move** | MIDI Tool and Audio Effect only | Opens chain_manager with processor highlighted; encoder 3 moves within chain, encoder 4 moves to another chain |
| **Replace** | All except MI/MR | Open engine selector to swap this processor with a different engine of the same type |
| **Remove** | MIDI Tool and Audio Effect | Remove this processor from the chain |
| **Randomize parameters** | MIDI Synth only | Set all parameters to random values |
| **Undo Randomize** | After randomize only | Restore previous values before the randomize |
| **Clean MIDI-learn** | Always | Remove CC bindings from all parameters in this processor |
| **Info** | Always | Show engine information: name, version, capabilities |
| **Insert MIDI Processor** | MIDI Synth or MIDI Tool | Insert a new MIDI processor at this position |
| **Insert Audio Processor** | MIDI Synth, Audio Effect, Audio Generator | Insert a new audio processor at this position |

**Move detail:** in the chain manager, encoder 3 nudges the processor left/right within the same signal chain (changing serial vs parallel order). Encoder 4 moves the processor to a different chain entirely.

Source: [`zyngui/zynthian_gui_processor_options.py`](../zynthian-ui/zyngui/zynthian_gui_processor_options.py)

---

## MIDI Config Screen

The `midi_config` subscreen shows all detected MIDI ports with routing status for the selected chain.

### Input Mode Icons

Each listed input port may show icons before its name:

| Icon | Meaning |
|------|---------|
| `⇥` | Active mode — this port drives only the active chain |
| `⇶` | Multitimbral mode — this port drives all chains by channel |
| `♣` | Sequencer capture — this port feeds the step sequencer |
| `⏱` | MIDI Clock — this port provides the tempo clock |
| `⌨` | Controller driver loaded — a ctrldev driver handles this device |

**Toggle a port:** tap to connect/disconnect from the active chain (shown as ☑/☐).

**Bold press a port:** opens an options menu for that port (change mode, set as clock source, etc.).

Source: [`zyngui/zynthian_gui_midi_config.py`](../zynthian-ui/zyngui/zynthian_gui_midi_config.py)

---

## Audio Routing

### audio_out Screen

Shows all available JACK output endpoints. By default, chains feed the main stereo mix. Change this to route a chain to a mixbus or a separate output pair.

| Action | Result |
|--------|--------|
| Tap a port | Toggle this output as a destination for the chain |
| ☑ port | Chain currently feeds that output |
| ☐ port | Chain does not feed that output |

### audio_in Screen

For Audio Input, MIDI+Audio, and Mixbus chains — selects which JACK input ports feed this chain. Used to route a microphone or USB audio input into an effects chain.

Both screens are accessed from chain_control: show the side chain panel → navigate to the audio routing node → tap to open the subscreen.

---

## Signal Flow Within a Chain

```
MIDI input (ZynMidiRouter)
    ↓
[MIDI-FX processors] (zero or more, in series or parallel)
    ↓
Synth engine / Audio Generator
    ↓
[Audio-FX processors] (zero or more, in series or parallel)
    ↓
ZynMixer → JACK output
```

**Serial vs parallel processors:** in the chain manager graph, processors in the same vertical column are parallel (receive the same signal). Processors in consecutive rows are serial (signal passes through each in turn). You can arrange them by using the Move option in processor_options.

---

## Example: Keyboard Split — Piano Low / Strings High

Two chains, both on MIDI channel 1, split at C4:

1. Add Instrument chain → FluidSynth → "Piano" bank → "Steinway Grand".
2. chain_options → note range → set **Max** to B3 (MIDI note 59).
3. Add second Instrument chain → ZynAddSubFX → "Strings" bank → any strings preset.
4. chain_options → note range → set **Min** to C4 (MIDI note 60).
5. Both respond to channel 1. ZynMidiRouter delivers notes to the matching chain based on pitch.

---

## Example: Serial Audio Chain — Synth + Reverb + Compressor

Add effects to an existing FluidSynth chain:

1. Select FluidSynth chain in chain_manager.
2. chain_options → **Add Audio-FX processor** → LV2 Plugin → "Calf Reverb".
3. chain_options → **Add Audio-FX processor** → LV2 Plugin → "Calf Multiband Compressor".
4. The chain graph now shows: FluidSynth → Calf Reverb → Calf Compressor → output.
5. Enter each effect's control screen to adjust its parameters.
6. Use processor_options → **Move** to reorder if needed (compressor before reverb for different character).

---

## Example: Export Chain for Reuse

Save the organ chain as a reusable snapshot:

1. Select the setBfree chain.
2. chain_options → **Export chain as snapshot** → choose a folder → type a name.
3. A `.zss` file is created in `/zynthian/zynthian-my-data/snapshots/<folder>/`.
4. In a future session, Main Menu → Snapshots → Import → navigate to that `.zss` to load just the organ chain into the current setup.

---

## What's Next

- [Control Screen](control-screen.html) — adjust engine parameters
- [MIDI CC Learning](midi-cc-learn.html) — bind physical knobs to parameters
- [Synth Engines](synth-engines.html) — which engine to use and why
- [Performance & CPU](performance-monitoring.html) — how many chains you can run

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_add_chain.py`, `zyngui/zynthian_gui_chain_manager.py`, `zyngui/zynthian_gui_chain_options.py`, `zyngui/zynthian_gui_processor_options.py`, `zyngui/zynthian_gui_chain_control.py`, `zyngui/zynthian_gui_midi_config.py`.*
