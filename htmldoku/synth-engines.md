# Synth Engines

Zynthian ships with a large collection of synthesis engines and audio processors. Each engine is a separate program managed by the `zyngine` layer [`zynthian-ui/zyngine/zynthian_engine_*.py`]. This page lists the available engines and explains when to use each one.

---

## How Engines Work

When you add a chain and select an engine, Zynthian starts the engine as a subprocess, connects it to JACK audio, and exposes its presets in the UI. The engine runs until the chain is removed or the snapshot changes. Multiple chains can run different engines simultaneously, subject to CPU limits.

Presets are organized into **banks**. A bank is a folder of preset files on disk — for FluidSynth, a `.sf2` soundfont file; for ZynAddSubFX, a folder of `.xmz` patches.

---

## Native Synthesizers

### ZynAddSubFX
**Type:** Polyphonic additive/subtractive/pad synthesizer.
**When to use:** Rich orchestral pads, evolving textures, complex timbres. CPU-intensive on large polyphony counts.
**Engine file:** `zynthian_engine_zynaddsubfx.py` [`zynthian-ui/zyngine/zynthian_engine_zynaddsubfx.py`].
Presets cover hundreds of instruments from strings and brass to electronic and experimental sounds.

### FluidSynth
**Type:** SF2 soundfont sampler.
**When to use:** Realistic acoustic instruments from General MIDI soundfonts, pianos, strings, drums.
**Engine file:** `zynthian_engine_fluidsynth.py` [`zynthian-ui/zyngine/zynthian_engine_fluidsynth.py`].
Zynthian ships with several soundfonts; additional `.sf2` files can be uploaded via webconf.

### setBfree
**Type:** Tonewheel organ emulator (Hammond B3/C3).
**When to use:** Organ sounds with rotating speaker (Leslie) simulation.
**Engine file:** `zynthian_engine_setbfree.py` [`zynthian-ui/zyngine/zynthian_engine_setbfree.py`].
Drawbar settings are exposed as MIDI controllers.

### LinuxSampler
**Type:** SFZ/GIG sampler — high-quality sample playback.
**When to use:** Professional orchestral libraries, Gigasampler (`.gig`) and SFZ format instruments.
**Engine file:** `zynthian_engine_linuxsampler.py` [`zynthian-ui/zyngine/zynthian_engine_linuxsampler.py`].

### Sfizz
**Type:** SFZ sampler.
**When to use:** SFZ format instruments, a lighter alternative to LinuxSampler.
**Engine file:** `zynthian_engine_sfizz.py` [`zynthian-ui/zyngine/zynthian_engine_sfizz.py`].

### Aeolus
**Type:** Pipe organ simulator.
**When to use:** Authentic pipe organ sounds.
**Engine file:** `zynthian_engine_aeolus.py` [`zynthian-ui/zyngine/zynthian_engine_aeolus.py`].

### Pianoteq
**Type:** Physical modelling piano (trial version included; commercial license available).
**When to use:** High-quality piano with physical modelling — better CPU/memory ratio than sample-based pianos.
**Engine file:** `zynthian_engine_pianoteq.py` [`zynthian-ui/zyngine/zynthian_engine_pianoteq.py`].

### Audio Player (ZynSampler)
**Type:** Audio file player.
**When to use:** Playing back audio clips (WAV, FLAC, MP3) as part of a live setup.
**Engine file:** `zynthian_engine_audioplayer.py` [`zynthian-ui/zyngine/zynthian_engine_audioplayer.py`].

---

## LV2 Plugins (via Jalv)

LV2 plugins are the largest category. Zynthian runs them through a customized Jalv host [`zynthian-ui/zyngine/zynthian_engine_jalv.py`]. They appear in the engine selector grouped by type.

Notable LV2 synths available in ZynthianOS:
- **Dexed** — Yamaha DX7 FM synthesizer emulator
- **OB-Xd** — Oberheim OB-X analog emulator
- **Vitalium** — Wavetable synthesizer (open-source Vital)
- **Surge XT** — Multi-synthesis with modulation engine
- **TAL NoizeMaker** — Virtual analog synthesizer
- **Odin 2** — Semi-modular synthesizer

LV2 effects (usable in effect chains):
- **Calf plugins** — Reverb, chorus, compressor, EQ, phaser
- **LSP plugins** — Professional dynamics and EQ
- **ZamaPlugins** — Audio dynamics processors

---

## Other Engines

### MOD-UI
Runs the MOD Devices plugin host with a browser-based pedalboard interface. Access via webconf or directly at port 8888. Best for complex effects chains.

### SooperLooper
Real-time audio looper — record and layer loops while playing.

### Pure Data
Run Pd patches as instruments or effects.

### MIDI Control / SysEx
Virtual engines for MIDI routing, control voltage, and SysEx message handling.

---

## Loading Presets

1. Tap an empty layer slot on the main screen (or press the Layer button on a Zynthian kit).
2. Select **Add Synth Engine** (or **Add Audio Effect**).
3. Browse and select an engine.
4. The bank list appears. Select a bank (soundfont, preset folder, or plugin category).
5. Select a preset.

The engine starts and MIDI notes on the selected channel now trigger that engine.

---

## What's Next

- [MIDI Controllers](midi.md) — connect a keyboard to play presets
- [Snapshots](snapshots.md) — save your engine/preset setup
- [Userguide](userguide.md) — how chains and layers work

---

*Version: 2026-05-25 — derived from `zynthian-ui/zyngine/zynthian_engine_*.py`.*
