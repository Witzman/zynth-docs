# Synth Engines

Zynthian ships with a large collection of synthesis engines and audio processors. Each engine is a separate program managed by the `zyngine` layer [`zynthian-ui/zyngine/zynthian_engine_*.py`]. This page lists the available engines, explains when to use each, and covers key parameters.

---

## How Engines Work

When you add a chain and select an engine, Zynthian starts the engine as a subprocess, connects it to JACK audio, and exposes its presets in the UI. The engine runs until the chain is removed or the snapshot changes. Multiple chains can run different engines simultaneously, subject to CPU limits.

Presets are organized into **banks**. A bank is a folder of preset files on disk — for FluidSynth, a `.sf2` soundfont file; for ZynAddSubFX, a folder of `.xmz` patches. Browse banks and presets from the chain's selector screen.

---

## ZynAddSubFX

**Type:** Polyphonic additive / subtractive / PAD synthesizer.
**Engine file:** `zynthian_engine_zynaddsubfx.py` [`zynthian-ui/zyngine/zynthian_engine_zynaddsubfx.py`]

The flagship Zynthian engine. Three synthesis modules work simultaneously within one patch: subtractive (SUB), additive (ADD), and PAD synthesis. Patches can combine all three for extremely complex, evolving timbres.

**Best for:** Rich strings, brass, pads, evolving textures, electronic sounds. The preset library covers hundreds of instruments from orchestral to experimental.

**Key parameters:**
| CC | Parameter |
|----|-----------|
| 1 | Modulation wheel — typically assigned to vibrato or filter mod |
| 7 | Volume |
| 11 | Expression |
| 74 | Filter cutoff (on most patches) |
| 71 | Filter resonance |

**CPU note:** ZynAddSubFX is CPU-intensive with large polyphony counts. On Pi 3, limit to 4–6 voices per patch. On Pi 4, 16+ voices are stable.

**Preset banks location:** `/zynthian/zynthian-data/presets/zynaddsubfx/`

---

## FluidSynth

**Type:** SF2 soundfont sampler.
**Engine file:** `zynthian_engine_fluidsynth.py` [`zynthian-ui/zyngine/zynthian_engine_fluidsynth.py`]

FluidSynth plays back sample-based instruments from SF2 soundfont files. Zynthian ships with several soundfonts including GeneralUser GS (128 General MIDI instruments + drums), and supports uploading custom `.sf2` files via webconf → Presets.

**Best for:** Realistic acoustic instruments, General MIDI playback, piano, strings, brass, drums, ethnic instruments.

**Key parameters:**
| CC | Parameter |
|----|-----------|
| 7 | Volume |
| 10 | Pan |
| 11 | Expression |
| 64 | Sustain pedal |
| 91 | Reverb send level |
| 93 | Chorus send level |

**Upload soundfonts:** webconf → Presets → upload `.sf2` file → appears as a new bank.

**Reverb/chorus:** Disabled by default. Enable via engine controls (navigate engine parameters, enable "Reverb Active" and set Room Size/Damping).

**SF2 files location:** `/zynthian/zynthian-my-data/soundfonts/sf2/`

---

## setBfree

**Type:** Tonewheel organ emulator (Hammond B3/C3 simulation with Leslie speaker).
**Engine file:** `zynthian_engine_setbfree.py` [`zynthian-ui/zyngine/zynthian_engine_setbfree.py`]

setBfree is a physically-modelled Hammond organ. It simulates all 91 tonewheels, the percussion (click), key click, preamp, and a full rotary speaker (Leslie) simulation.

**Best for:** Rock organ, gospel, jazz organ. The default setup is an excellent Hammond B3 with a rotating speaker.

**Key parameters:**
| CC | Parameter | Notes |
|----|-----------|-------|
| 1–9 | Drawbars (16', 5⅓', 8', 4', 2⅔', 2', 1⅗', 1⅓', 1') | 0=off, 127=full |
| 64 | Sustain pedal | Rotary speed fast/slow toggle |
| 11 | Expression (swell) | Volume swell pedal |

**MIDI channels:**
- Channel 1 → lower manual
- Channel 2 → upper manual
- Channel 3 → pedal bass

**Classic drawbar settings:**
- Full organ: `88 8888 888` (all drawbars at max)
- Gospel lead: `80 8800 000`
- Rock: `88 8800 000`
- Jazz mellow: `00 7800 000`

---

## LinuxSampler

**Type:** GIG and SFZ high-quality sampler.
**Engine file:** `zynthian_engine_linuxsampler.py` [`zynthian-ui/zyngine/zynthian_engine_linuxsampler.py`]

LinuxSampler plays Gigasampler (`.gig`) and SFZ format sample libraries. It is designed for professional sample libraries with round-robin, velocity layers, and release samples.

**Best for:** High-quality orchestral libraries (Sonatina, Versilian, Salamander Piano), commercial sample libraries.

**Key parameters:** Same as SF2 (CC 7 volume, CC 11 expression, CC 64 sustain). Specific articulation CCs depend on the library.

**Sample libraries location:** `/zynthian/zynthian-my-data/soundfonts/gig/` (GIG) or `/zynthian/zynthian-my-data/soundfonts/sfz/` (SFZ)

**CPU note:** LinuxSampler streams samples from disk. SD card read speed matters — use a quality card.

---

## Sfizz

**Type:** SFZ sampler (lighter alternative to LinuxSampler).
**Engine file:** `zynthian_engine_sfizz.py` [`zynthian-ui/zyngine/zynthian_engine_sfizz.py`]

Sfizz is a modern, efficient SFZ player. More compatible with extended SFZ 2.0 features than LinuxSampler, with lower CPU/memory overhead.

**Best for:** SFZ instruments, Salamander Piano, Versilian Studios VSCO, freely-available SFZ libraries.

**SFZ files location:** `/zynthian/zynthian-my-data/soundfonts/sfz/`

---

## Aeolus

**Type:** Pipe organ synthesizer (additive synthesis, not samples).
**Engine file:** `zynthian_engine_aeolus.py` [`zynthian-ui/zyngine/zynthian_engine_aeolus.py`]

Aeolus synthesizes pipe organ sounds from scratch using physically-modelled resonators. Unlike setBfree (Hammond), Aeolus sounds like a classical pipe organ — flutes, strings, principals, reeds.

**Best for:** Classical pipe organ, church organ, baroque organ. Distinct from setBfree — no rock organ sound.

**Key parameters:** Stop registration (which pipes are active) is set via MIDI program changes or the UI stop controls.

---

## Pianoteq

**Type:** Physical modelling piano (commercial — trial/licensed).
**Engine file:** `zynthian_engine_pianoteq.py` [`zynthian-ui/zyngine/zynthian_engine_pianoteq.py`]

Pianoteq models piano acoustics from first principles — string vibration, soundboard resonance, hammer felt, pedal mechanics. Produces a very realistic piano feel with much lower disk/RAM requirements than sample-based pianos.

**Best for:** Best-quality piano when a Pianoteq license is available. Trial version runs with limitations (some notes silenced). Commercial licenses available at [modartt.com](https://www.modartt.com/).

**Key parameters:** Pianoteq exposes ~50 physical parameters: hammer hardness, string length, soundboard resonance, unison width, etc. All are accessible via the engine control screen.

**Installation:** The `pianoteq` binary must be present at `/zynthian/zynthian-sw/pianoteq/`. The license file goes in the same directory.

---

## AudioPlayer (ZynSampler)

**Type:** Audio file player.
**Engine file:** `zynthian_engine_audioplayer.py` [`zynthian-ui/zyngine/zynthian_engine_audioplayer.py`]

Plays audio files (WAV, FLAC, OGG, MP3) from the Zynthian file system. Useful for backing tracks, sample pads, or live audio playback triggered by MIDI notes.

**Best for:** Background loops, sample playback, stem tracks for live performance.

**Files location:** `/zynthian/zynthian-my-data/audio/`

**Controls:** MIDI note triggers playback. File assignment to notes is configured in the engine's mapping screen.

---

## LV2 Plugins (via Jalv)

LV2 plugins are the largest category. Zynthian runs them through a customized Jalv host [`zynthian-ui/zyngine/zynthian_engine_jalv.py`]. They appear in the engine selector grouped by plugin type.

See [LV2 Plugins](lv2-plugins.md) for installation and management.

**Notable LV2 synths available in ZynthianOS:**

| Plugin | Type | Notes |
|--------|------|-------|
| Dexed | FM synthesizer | Yamaha DX7 emulator; 32-voice, very CPU-efficient |
| OB-Xd | Analog subtractive | Oberheim OB-X emulation |
| Vitalium | Wavetable | Open-source Vital; complex modulation |
| Surge XT | Multi-synthesis | Advanced FM/wavetable/subtractive + modulation |
| TAL NoizeMaker | Virtual analog | Simple, clean VA synth |
| Odin 2 | Semi-modular | With built-in sequencer/arpeggiator |

**Notable LV2 effects:**

| Plugin | Type | CC mapping |
|--------|------|-----------|
| Calf Reverb | Reverb | Room size, damping, wet/dry |
| Calf Compressor | Dynamics | Threshold, ratio, attack, release |
| ZaMultiCompX2 | Multiband comp | 3-band dynamics |
| LSP Parametric EQ | EQ | 8-band parametric |
| Guitarix | Amp sim | Guitar amp + cabinet emulation |

---

## MOD-UI

**Type:** MOD Devices plugin host with browser-based pedalboard.
**Engine file:** `zynthian_engine_modui.py` [`zynthian-ui/zyngine/zynthian_engine_modui.py`]

MOD-UI runs a pedalboard-style effects chain editor in a web browser. Access via `http://zynthian.local:8888` when the MOD-UI chain is active. Drag-and-drop LV2 plugins onto the pedalboard and connect them visually.

**Best for:** Complex effects chains, guitar processing, signal routing that would be tedious in the standard Zynthian UI.

---

## SooperLooper

**Type:** Real-time audio looper.
**Engine file:** `zynthian_engine_sooperlooper.py` [`zynthian-ui/zyngine/zynthian_engine_sooperlooper.py`]

SooperLooper records and layers live audio loops. Multiple loop slots can be stacked. See [Recipes](recipes.md) for a live looper setup walkthrough.

**Key MIDI control:**
| CC / Note | Function |
|-----------|----------|
| Toggle Record | CC 30 (configurable): starts recording; second press stops recording and starts playback |
| Toggle Play | Play/pause current loop |
| Undo | Remove last overdub |
| Mute | Mute/unmute loop |

---

## Pure Data

**Type:** Run Pd patches as instruments or effects.
**Engine file:** `zynthian_engine_puredata.py` [`zynthian-ui/zyngine/zynthian_engine_puredata.py`]

Pure Data patches are `.pd` files. Upload a patch to `/zynthian/zynthian-my-data/presets/puredata/` and it appears as a preset in the PureData engine. [low]

---

## MIDI Control / SysEx

**Engine file:** `zynthian_engine_midi_control.py`, `zynthian_engine_sysex.py` [`zynthian-ui/zyngine/`]

Virtual engines for MIDI routing and SysEx message handling. `MIDI Control` generates outgoing MIDI messages in response to chain controls. `SysEx` sends and receives SysEx from external hardware.

---

## Choosing an Engine

| Need | Engine |
|------|--------|
| Realistic piano | FluidSynth (Salamander SF2) or Pianoteq (licensed) |
| Hammond organ | setBfree |
| Pipe organ | Aeolus |
| Orchestral strings/brass | LinuxSampler or Sfizz (with Sonatina/Versilian) |
| FM synthesis (DX7) | Dexed (LV2) |
| Complex pads / textures | ZynAddSubFX |
| Wavetable / modern synth | Surge XT or Vitalium (LV2) |
| General MIDI playback | FluidSynth (GeneralUser GS soundfont) |
| Audio effects | Calf, LSP, or Guitarix (LV2) |
| Live looping | SooperLooper |
| Complex effects chain | MOD-UI |

---

## Loading Presets

1. Main screen → **+** (Add Chain).
2. Select **Add Synth Engine** (or **Add Audio Effect** for effects).
3. Browse and select an engine.
4. The bank list appears. Select a bank.
5. Select a preset.

The engine starts and MIDI notes on the selected channel trigger that engine.

---

## What's Next

- [LV2 Plugins](lv2-plugins.md) — installing and managing LV2 plugins
- [Recipes](recipes.md) — common multi-engine setups
- [MIDI Controllers](midi.md) — connect a keyboard to play presets
- [Snapshots](snapshots.md) — save your engine/preset setup

---

*Version: 2026-05-25 — derived from `zynthian-ui/zyngine/zynthian_engine_*.py`.*
