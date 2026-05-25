# Common Setups (Recipes)

Step-by-step walkthroughs for the most useful Zynthian configurations. Each recipe names the engines and settings involved.

---

## Recipe 1 — Layered Piano + Strings

**Goal:** Play a piano sound on the bottom half of a keyboard, strings on the top half, both on the same MIDI channel. Classic split-layer setup.

**Engines needed:** FluidSynth (piano), ZynAddSubFX or LinuxSampler (strings).

**Steps:**

1. Start Zynthian. If any chains exist, Admin → Clean All.
2. Main screen → **+** (Add Chain) → select **FluidSynth**.
3. In FluidSynth: load a soundfont (e.g. `GeneralUser GS` or `Yamaha C5 Grand`). Select the Acoustic Piano preset.
4. Add a second chain: Main screen → **+** → select **ZynAddSubFX**.
5. In ZynAddSubFX: navigate to Bank → select `Strings` → load any string ensemble preset.
6. Set both chains to MIDI channel 1 (or Omni).
7. For the split: tap chain 1 (Piano) → options → **Note Range** → set High Note to B3 (MIDI note 59).
8. Tap chain 2 (Strings) → options → **Note Range** → set Low Note to C4 (MIDI note 60).
9. Play — piano on lower half, strings on upper.
10. Save: Admin → Snapshots → Save As → name it `Piano+Strings`.

**Tips:** Adjust relative volumes with the mixer (Main → Mixer, or encoder 4 on V5).

---

## Recipe 2 — Hammond Organ with Rotary Speaker

**Goal:** Full Hammond B3 organ with Leslie rotary speaker simulation. The classic rock/jazz organ.

**Engines needed:** setBfree.

**Steps:**

1. Add Chain → select **setBfree**.
2. setBfree loads with a default organ preset. The drawbars are the 9 fundamental registers (16', 5⅓', 8', 4', 2⅔', 2', 1⅗', 1⅓', 1').
3. Navigate engine parameters (encoder 3 on V5, or tap the engine name):
   - **Drawbars**: control tone color. Classic rock setting: `888000000`.
   - **Rotary**: enable Leslie speaker simulation.
   - **Rotary Speed**: toggle between Slow/Fast (CC 64 = sustain pedal, or assign a button).
4. For authentic expression: CC 11 (Expression) controls the volume swell. Assign your controller's expression pedal to CC 11.
5. For rotary on/off with foot switch: webconf → MIDI → CC → map CC 64 to `Rotary Toggle`.

**Tips:** setBfree responds on all 16 MIDI channels simultaneously (each channel = one manual). Channel 1 = lower manual, channel 2 = upper manual, channel 3 = pedal bass.

---

## Recipe 3 — Live Looper

**Goal:** Record a live audio loop and layer on top of it. Useful for solo performance.

**Engines needed:** SooperLooper + any instrument engine.

**Steps:**

1. Add your instrument chain (e.g. FluidSynth piano on MIDI channel 1).
2. Add a second chain → select **SooperLooper**.
3. SooperLooper chain captures audio from the instrument chain's output. Set its audio input to the instrument's JACK output in webconf → JACK routing, or use the Zynthian audio mixer to route it.
4. Assign record/play/stop controls:
   - webconf → MIDI → CC → map a CC to `SooperLooper Record` (starts recording on first press, stops on second press, starts playback).
   - Alternatively: on V5 hardware, switch 9 short-press = TOGGLE_RECORD by default.
5. Performance flow:
   - Play your instrument.
   - Press Record → play a phrase → press Record again (overdub mode starts).
   - Press Play to loop without overdub.
   - Press Undo to remove last overdub.
   - Press Stop → Clear to reset.

**Tips:** SooperLooper can run multiple loop slots. Each slot = a separate `+ chain` instance.

---

## Recipe 4 — Multi-Engine Drum Kit

**Goal:** Drums from FluidSynth (or LinuxSampler) on channel 10, bass on channel 2, lead synth on channel 3. Standard General MIDI layout.

**Engines needed:** FluidSynth (drums + bass), ZynAddSubFX (lead).

**Steps:**

1. Add Chain → FluidSynth → load a General MIDI soundfont (e.g. `FluidR3_GM.sf2`).
2. Set this chain to MIDI channel 10.
3. In FluidSynth, navigate to Bank 128 (percussion) → select `Standard Drum Kit`.
4. Add second chain → FluidSynth → same soundfont.
5. Set to MIDI channel 2 → navigate to Acoustic Bass or Electric Bass preset.
6. Add third chain → ZynAddSubFX → set to MIDI channel 3 → load a lead synth preset.
7. From a DAW or sequencer, send:
   - Drums on MIDI ch 10
   - Bass on MIDI ch 2
   - Lead on MIDI ch 3
8. Save as snapshot.

**Tips:** Use webconf → MIDI → Ports to confirm your DAW's MIDI output port is active.

---

## Recipe 5 — Headless Operation (No Display)

**Goal:** Run Zynthian with only SSH/webconf access — no monitor needed.

**Steps:**

1. Connect Pi to your network via Ethernet (most reliable) or configure WiFi via webconf first.
2. In webconf → Hardware → Wiring: set layout to `TOUCH_ONLY`. This disables GPIO encoder scanning.
3. All control is via `http://zynthian.local`:
   - Load presets from Presets page.
   - Load/save snapshots from Snapshots page.
   - Adjust audio settings as usual.
4. MIDI controllers still work over USB.
5. Optional: enable VNC in webconf → System → VNC for a remote desktop view of the Zynthian UI.

**Tips:** `ssh root@zynthian.local` always works. The webconf auto-starts on every boot.

---

## Recipe 6 — LV2 Plugin Chain (Effects)

**Goal:** Add a reverb or compressor LV2 effect to a synth chain.

**Engines needed:** Any synth + Jalv (LV2 host).

**Steps:**

1. Add your instrument chain (e.g. ZynAddSubFX).
2. Add a second chain → **Jalv** → navigate to the LV2 plugin list.
3. Find a reverb plugin (e.g. `Calf Reverb`, `ZaMultiCompX2`, `LSP Reverb`).
4. Load it. The plugin appears as a chain with JACK audio ports.
5. Route the instrument chain's output to the LV2 chain's input via the Zynthian audio mixer or JACK routing.

> **Note:** LV2 plugin availability depends on what is installed on the OS image. Run `ls /usr/lib/lv2/` to see installed plugins. Use webconf → Engines to regenerate the plugin cache after installing new ones. [low]

**Tips:** Install additional LV2 plugins with `apt install lv2-*` via SSH and regenerate cache.

---

## Recipe 7 — MIDI Recording

**Goal:** Record a MIDI performance and play it back.

**Steps:**

1. Ensure a MIDI controller is connected and a chain is active.
2. On the Zynthian screen: navigate to **MIDI Recorder** (Admin → MIDI Recorder, or switch 11 bold on V5).
3. Press Record. Play your performance.
4. Press Stop.
5. Press Play to hear it back — the MIDI file plays through the active chains.
6. Save: MIDI Recorder → Save. File goes to `/zynthian/zynthian-my-data/midi/`.

**Tips:** Transfer recordings via SFTP to your computer: `sftp root@zynthian.local`, navigate to `/zynthian/zynthian-my-data/midi/`.

---

## What's Next

- [Synth Engines](synth-engines.md) — full engine reference
- [Snapshots](snapshots.md) — save any recipe as a snapshot
- [Troubleshooting](troubleshooting.md) — when something doesn't work

---

*Version: 2026-05-25*
