# Chains & Routing

A **chain** is the core unit of Zynthian: one MIDI input → one or more processors → one audio output. This page covers how to create, configure, and route chains.

---

## Chain Types

When you add a chain, you choose its type. Each type sets what kind of signal flows through it.

| Type | MIDI | Audio in | Audio out | Use for |
|------|------|----------|-----------|---------|
| **Instrument** | Yes | No | Yes | Synth engines: FluidSynth, ZynAddSubFX, setBfree |
| **Audio Input** | No | From interface | Yes | Live audio with effects |
| **Clip Launcher** | No | No | Yes | Play back audio clips / loops |
| **MIDI** | Yes (thru) | No | No | MIDI processing tools |
| **MIDI + Audio** | Yes (thru) | Yes (thru) | Yes | Combined MIDI/audio processor |
| **Audio Generator** | No | No | Yes | Tone generators, file playback |
| **Special** | Yes (thru) | Yes (thru) | Yes | Catch-all for special engines |
| **Mixbus** | No | Yes (thru) | Yes | Group bus / send effects |

Source: [`zyngui/zynthian_gui_add_chain.py:42`](../zynthian-ui/zyngui/zynthian_gui_add_chain.py)

> Clip Launcher is limited to 16 instances. The button is grayed out if this limit is reached.

---

## Adding a Chain

**From the mixer:** tap the **+** button (rightmost empty strip or via chain_manager).

**From chain_manager:** tap **Add chain** button.

The flow:

```
add_chain (pick type)
    → engine category list
        → engine list
            → bank list
                → preset list
                    → control screen
```

1. **Pick chain type** — Instrument is the most common.
2. **Pick engine category** — e.g. "Sound Synthesis", "LV2 Plugin", "Special".
3. **Pick engine** — e.g. FluidSynth, ZynAddSubFX, setBfree, or a specific LV2 plugin.
4. **Pick bank** — a collection of related presets (e.g. "Piano", "Strings").
5. **Pick preset** — the specific sound (e.g. "Steinway Grand").
6. **Control screen opens** — ready to play.

---

## Chain Manager

Access: Main Menu → **Chain Manager**, or SW1 bold press from chain_control.

Shows a visual graph of all chains as columns. Each column has:
- Chain name at the top
- Processor nodes stacked vertically (MIDI-FX → synth engine → Audio-FX)
- Connections between nodes shown as lines

**Selecting in chain_manager:**

| Action | Result |
|--------|--------|
| Tap chain column header | Select that chain |
| Tap processor node | Enter chain_control for that processor |
| Long-press chain header | Chain options |
| Encoder 3 | Scroll between chains |

**Moving a chain:** chain_options → **Move chain** — encoder 4 then repositions the chain left/right. Confirm with select.

Source: [`zyngui/zynthian_gui_chain_manager.py`](../zynthian-ui/zyngui/zynthian_gui_chain_manager.py)

---

## Chain Control

The `chain_control` screen is a compound view: a **side chain graph** on the left plus a **subscreen** on the right. The subscreen can show:

| Subscreen | What it shows |
|-----------|--------------|
| `control` | Parameter knobs for the current processor |
| `chain_options` | Options for the current chain |
| `midi_config` | MIDI input / output routing |
| `audio_out` | Audio output routing |
| `audio_in` | Audio input routing |

Toggle the side chain panel with **SW3 short** or by tapping the chain edge.

When the side chain is visible, Encoder 2 navigates between processor nodes in the graph. Tap a node to switch the subscreen to that processor's controls.

Source: [`zyngui/zynthian_gui_chain_control.py`](../zynthian-ui/zyngui/zynthian_gui_chain_control.py)

---

## Chain Options

Long-press a chain in the chain_manager, or from chain_control → options subscreen.

| Option | What it does |
|--------|-------------|
| Clear MIDI Learn | Remove all CC bindings from all processors in this chain |
| Rename chain | Type a new name (keyboard screen) |
| Move chain | Reposition in mixer (encoder 4) |
| Add MIDI-FX processor | Append a MIDI effect at the end of the MIDI chain |
| Add Audio-FX processor | Append an audio effect at the end of the audio chain |
| Remove all MIDI-FX | Strip all MIDI effect processors |
| Remove all Audio-FX | Strip all audio effect processors |
| Remove chain | Delete chain and all its processors |
| Export chain as snapshot | Save this chain configuration as a `.zss` file |
| Insert new chain | Create a new chain immediately before this one |
| Remove ALL | Remove all chains / sequences (with confirmation) |

Source: [`zyngui/zynthian_gui_chain_options.py`](../zynthian-ui/zyngui/zynthian_gui_chain_options.py)

---

## Audio Routing

By default, each chain's audio output goes to the main stereo mix. You can override this.

**audio_out screen:** select which JACK ports this chain feeds. Use for routing a chain to a mixbus instead of main out.

**audio_in screen:** for Audio Input and MIDI+Audio chains — select which JACK input ports feed into this chain.

Both screens are accessible from chain_control → tap the chain graph edge to open the side panel → navigate to the routing node.

---

## MIDI Config

The `midi_config` subscreen shows MIDI input and output assignments for the chain:

- **MIDI input:** which MIDI channels / ports this chain listens to
- **MIDI output:** which ports this chain's MIDI thru sends to

Accessible from chain_control → side chain panel → MIDI input/output node.

---

## Example: Keyboard Split — Piano Low / Strings High

Two chains, both on MIDI channel 1, divided at C4:

1. Add Instrument chain → FluidSynth → "Piano" bank → "Steinway Grand" preset.
2. From chain_control → chain_options → **Note Range** → set Max to B3.
3. Add a second Instrument chain → ZynAddSubFX → "Strings" bank → any strings preset.
4. chain_options → **Note Range** → set Min to C4.

Both chains receive channel 1. ZynMidiRouter delivers notes to whichever chain's note range matches.

---

## Example: Serial Audio Chain — Synth + Reverb

Add a reverb LV2 as an audio-FX on an existing instrument chain:

1. Select the FluidSynth chain in chain_manager.
2. chain_options → **Add Audio-FX processor**.
3. Engine category: LV2 Plugin → "Calf Reverb" (or any other reverb LV2).
4. The processor appears as a new node after the synth in the chain graph.
5. Control screen for the reverb shows Calf Reverb parameters (Room size, Diffusion, etc.).

The signal flow becomes: FluidSynth → Calf Reverb LV2 → main output.

---

## What's Next

- [Control Screen](control-screen.html) — adjust engine parameters
- [MIDI CC Learning](midi-cc-learn.html) — bind physical knobs to parameters
- [Synth Engines](synth-engines.html) — which engine to use

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_add_chain.py`, `zyngui/zynthian_gui_chain_manager.py`, `zyngui/zynthian_gui_chain_options.py`, `zyngui/zynthian_gui_chain_control.py`.*
