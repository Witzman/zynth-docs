# Glossary

Key terms used throughout this documentation.

---

## A

**ALSA** — Advanced Linux Sound Architecture. The Linux kernel's audio subsystem. Zynthian uses ALSA as the hardware interface below JACK. Card names like `hw:S2` are ALSA identifiers.

**Aeolus** — Software pipe organ engine. Simulates organ pipes with additive synthesis. See [Synth Engines](synth-engines.md).

**aconnect** — ALSA command-line tool to list and connect MIDI ports. `aconnect -l` shows all available MIDI clients.

**AudioPlayer** — Zynthian engine for playing audio files (WAV, MP3, OGG) from a chain.

---

## B

**Bank** — A collection of presets grouped together within an engine. For FluidSynth, a bank is a MIDI bank number within a soundfont. For ZynAddSubFX, a bank is a folder of `.xiz` preset files.

**BLE MIDI** — Bluetooth Low Energy MIDI. The MIDI profile used by wireless MIDI controllers. Pairs via `bluetoothctl`. See [MIDI Controllers](midi.md).

**Buffer size** — Number of audio frames JACK processes per cycle. Lower buffer = lower latency + higher CPU. Higher buffer = more stable + higher latency. Typical values: 128, 256, 512 frames.

---

## C

**Chain** — One engine instance with its MIDI input channel and audio output assignment. You can run multiple chains simultaneously (e.g. piano on channel 1, organ on channel 2). Managed by `zynthian_chain_manager.py`.

**CC** — MIDI Control Change. 7-bit messages (CC 0–127) used to send continuous controller data (knob turns, pedal position, modulation wheel). Webconf lets you map CC numbers to Zynthian parameters.

---

## D

**DIN-5 MIDI** — The classic 5-pin MIDI connector. Available on official Zynthian kits (V5, Z2). Must be enabled in webconf → Hardware → MIDI Options on custom builds.

---

## E

**Engine** — A synthesizer or audio processor that Zynthian loads into a chain. Examples: ZynAddSubFX, FluidSynth, setBfree, Jalv (LV2), LinuxSampler. See [Synth Engines](synth-engines.md).

---

## F

**FluidSynth** — Software MIDI synthesizer that plays SF2 soundfonts. Good for realistic piano, strings, drums. See [Synth Engines](synth-engines.md).

---

## H

**HifiBerry DAC+** — A popular I2S audio HAT for Raspberry Pi. Supported natively by Zynthian; detected by `zynthian_autoconfig.py`.

---

## I

**I2S** — Inter-IC Sound. Digital audio interface used by audio HATs (HifiBerry, ZynADAC) connected to the Pi's GPIO header.

---

## J

**JACK** — JACK Audio Connection Kit. Low-latency audio server used by Zynthian to route audio between engines and the output device. Runs as `jack2.service`. Must be running before Zynthian can produce sound.

**Jalv** — LV2 plugin host. Zynthian uses it to run any LV2 plugin as an engine. The `zynthian_engine_jalv.py` wrapper starts jalv as a subprocess.

---

## L

**Layer** — Older Zynthian terminology for what is now called a "chain". The UI may still use "layer" in some places.

**Latency** — Time between playing a note and hearing sound. Measured in milliseconds. Determined by buffer size ÷ sample rate. At 256 frames / 44100 Hz ≈ 5.8ms.

**last_state.zss** — Special snapshot file. If present in the snapshots directory, Zynthian loads it automatically on startup. See [Snapshots](snapshots.md).

**libzyncore.so** — Compiled shared library from the `zyncoder` C project. Required by both `zynthian-ui` and `zynthian-webconf`. If missing, neither service can start.

**LinuxSampler** — High-quality sampler engine. Supports GIG and SFZ format sample libraries.

**LV2** — A standard plugin format for Linux audio. Many synthesizers and effects are available as LV2 plugins. Zynthian runs them via Jalv.

---

## M

**mDNS** — Multicast DNS. Allows `zynthian.local` hostname resolution without a static IP. Requires Bonjour (Windows) or Avahi (Linux).

**MOD-UI** — Web-based pedalboard interface from MOD Devices. Zynthian includes a `modui` engine type.

---

## O

**Omni** — MIDI channel mode where a chain responds to all 16 MIDI channels instead of just one.

---

## P

**PC** (Program Change) — MIDI message for switching presets. PC 0–127 selects a preset within the current bank.

**Pianoteq** — Commercial physical modeling piano plugin. Zynthian supports it if you have a license. See [Synth Engines](synth-engines.md).

**Preset** — A saved configuration of an engine's parameters. Loading a preset switches timbre/sound. Presets live inside banks.

---

## Q

**QMidiNet** — Network MIDI protocol. Lets multiple devices share MIDI over a LAN. Enable in webconf → MIDI → Network.

---

## S

**Sample rate** — Number of audio samples per second. Common values: 44100 Hz (CD standard), 48000 Hz (professional standard). Must match what your audio device supports.

**setBfree** — Tonewheel organ emulator. Simulates the Hammond B3 organ with rotary speaker simulation. See [Synth Engines](synth-engines.md).

**SF2** — SoundFont 2. File format for sample-based instruments (.sf2 files). Loaded by FluidSynth.

**SFZ** — Open sample format. Used by LinuxSampler and Sfizz.

**Sfizz** — SFZ sample player. See [Synth Engines](synth-engines.md).

**Snapshot** — A complete saved state of Zynthian: all chains, engines, presets, MIDI assignments, mixer levels. Stored as a `.zss` file. See [Snapshots](snapshots.md).

**SooperLooper** — Audio looper engine. Records and plays back live audio loops.

---

## U

**USB Audio** — Audio device connected via USB. Plug-and-play on Linux. Selected by card name (`hw:S2`) in JACK configuration.

---

## V

**V5** — The current official Zynthian hardware kit. Raspberry Pi 4 + ZynADAC audio codec + TPA6130 headphone amp + 3.5" touchscreen + 4 rotary encoders + MIDI DIN.

---

## W

**Webconf** — The web configuration interface at `http://zynthian.local`. Used to configure audio, MIDI, display, and perform software updates. See [Webconf Reference](webconf.md).

**Wiring profile** — A configuration that maps GPIO pins to physical controls (encoders, buttons, LEDs). `TOUCH_ONLY` is used for setups with no physical encoders (mouse/touchscreen only).

---

## X

**XRUN** — An audio buffer underrun or overrun in JACK. Sounds as a click or glitch. Caused by CPU overload or too-small buffer size. Monitor with `journalctl -u jack2 -f | grep -i xrun`.

---

## Z

**Z2** — An older official Zynthian hardware kit. Raspberry Pi 4 + ZynADAC + touch display.

**ZynADAC** — Official Zynthian audio codec board. Uses PCM5242 (DAC) + PCM1863 (ADC) chips. Detected automatically by `zynthian_autoconfig.py`.

**ZynAddSubFX** — The flagship Zynthian synthesizer engine. Additive + subtractive + FM synthesis with a large preset library. See [Synth Engines](synth-engines.md).

**zyncoder** — C library compiled into `libzyncore.so`. Handles hardware encoder reading, MIDI routing, and chain management at the native layer.

**ZynMidiRouter** — Native C module (part of zyncoder) that routes MIDI between inputs and engines.

**zynthian-sys** — The system repository. Contains boot scripts, systemd units, hardware config. See [Architecture](architecture.md).

**zynthian-ui** — The UI repository. Contains the Zynthian application (GUI + engines).

**zynthian-webconf** — The webconf repository. Contains the web configuration interface.

**ZynScreen** — Official Zynthian 3.5" touchscreen display.

**ZSS** — Zynthian Snapshot (file extension). Human-readable YAML format. See [Snapshots](snapshots.md).

---

*Version: 2026-05-25*
