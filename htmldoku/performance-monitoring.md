# Performance & CPU Management

Running multiple synth engines simultaneously on a Raspberry Pi requires attention to CPU budget. This page explains how to read performance indicators, diagnose problems, and tune your setup for stable, glitch-free operation.

---

## Status Bar CPU Indicator

The status bar at the top of every screen shows a percentage. This is the **total system CPU load** averaged across all cores.

| Range | Meaning | Recommended action |
|-------|---------|-------------------|
| 0–50% | Comfortable headroom | None |
| 50–70% | Moderate | Monitor; avoid adding heavy engines |
| 70–85% | High | Risk of XRUNs under peak MIDI load |
| 85–100% | Critical | Audio glitches likely; reduce load now |

**Caveat:** on Pi 4 (4 cores), a single core saturated at 100% shows as ~25% total. If one engine is bottlenecking, CPU% alone can be misleading. Use `htop` over SSH to see per-core and per-process breakdown:

```bash
ssh root@zynthian.local "htop"
```

---

## DPM — Digital Peak Meters

The mixer shows vertical level meters for each chain (DPM = Digital Peak Meter). These are **pre-clip indicators** — the top red zone means the signal is at or above 0 dBFS.

| Meter zone | Level | Action |
|------------|-------|--------|
| Green (lower 2/3) | −∞ to −6 dBFS | Safe |
| Yellow (upper 1/3) | −6 to −1 dBFS | Approaching clip |
| Red (top 1–2 bars) | −1 to 0 dBFS | Near clipping — reduce engine volume |
| Solid red | 0 dBFS (clipping) | Distortion — reduce immediately |

**Enable/disable DPM:** Admin → AUDIO → **Mixer Peak Meters** → toggle. Env var: no direct env var — stored in `ZYNTHIAN_UI_ENABLE_DPM` by `zynconf.save_config()`. Disabling saves ~1–2% CPU, useful on tight Pi 3 setups.

The master chain DPM (far right strip) shows the final summed output level. If it clips while individual chains do not, reduce the master fader or lower multiple chain volumes simultaneously.

---

## JACK XRUN Indicator

An **XRUN** (buffer under-run or over-run) occurs when the CPU cannot process an audio period in time. The result is an audible click or dropout in the audio output.

The status bar shows an XRUN counter (small ✕ or X with count). If this increments during playback, the CPU cannot keep up.

### Interpreting XRUN Timing

| XRUNs occur | Likely cause |
|-------------|-------------|
| During heavy chord or many simultaneous notes | Polyphony overload on ZynAddSubFX or similar |
| On MIDI note-on only | Engine voice startup cost too high |
| Randomly, not correlated to MIDI | Thermal throttling or background OS task |
| Consistently at specific patterns | Pattern requires sustained CPU above system capability |
| Only during patch changes | Engine reload is heavy — use ZS3 instead of snapshots during performance |

### Checking JACK Status

```bash
ssh root@zynthian.local "systemctl status jack2 --no-pager"
ssh root@zynthian.local "journalctl -u jack2 -n 20 --no-pager"
```

JACK log will show XRUN messages with timestamps.

---

## JACK Configuration Parameters

The JACK audio server is configured via `JACKD_OPTIONS` in `zynthian_envars.sh`:

```bash
JACKD_OPTIONS="-P 70 -t 2000 -s a -d alsa -d hw:S2 -r 44100 -p 256 -n 3"
```

| Flag | Value | Meaning |
|------|-------|---------|
| `-P` | 70 | JACK process real-time priority (0–99, higher = more priority) |
| `-t` | 2000 | Client timeout in ms before JACK kills it |
| `-s a` | alsa | Use ALSA audio backend |
| `-d alsa` | — | ALSA driver |
| `-d hw:S2` | hw:S2 | ALSA hardware device name (S2 = Zynthian soundcard) |
| `-r` | 44100 | Sample rate in Hz |
| `-p` | 256 | Frames per period (buffer size) |
| `-n` | 3 | Number of periods per cycle |

### Buffer Size and Latency

| `-p` value | Latency at 44100 Hz | XRUN risk | Notes |
|-----------|--------------------|-----------|----|
| 128 | 2.9 ms | High | Only on Pi 4/5 with simple patches |
| 256 | 5.8 ms | Low | Standard — good balance |
| 512 | 11.6 ms | Very low | Use if XRUNs persist with 256 |
| 1024 | 23.2 ms | Minimal | Noticeable latency; last resort |

Change buffer size in webconf → **Audio** → **Period Size**. Takes effect after restart.

---

## Temperature Monitoring

Raspberry Pi 4 throttles CPU at **80°C** and hard-throttles at **85°C**, both causing XRUNs and UI sluggishness.

### Check Temperature

```bash
ssh root@zynthian.local "vcgencmd measure_temp"
# → temp=57.3'C
```

### Check Throttle State

```bash
ssh root@zynthian.local "vcgencmd get_throttled"
```

The returned value is a bitmask. Decode it:

| Bit position | Hex mask | Meaning |
|-------------|----------|---------|
| 0 | 0x1 | Under-voltage detected (now) |
| 1 | 0x2 | Arm frequency capped (now) |
| 2 | 0x4 | Currently throttled |
| 3 | 0x8 | Soft temperature limit active |
| 16 | 0x10000 | Under-voltage occurred since boot |
| 17 | 0x20000 | Arm frequency capped since boot |
| 18 | 0x40000 | Throttling occurred since boot |
| 19 | 0x80000 | Soft temperature limit since boot |

Common responses:

| Value | Meaning |
|-------|---------|
| `throttled=0x0` | No throttling — healthy |
| `throttled=0x50000` | Was throttled and had under-voltage at some point since boot, but OK now |
| `throttled=0x50005` | Currently throttled AND under-voltage — check power supply and airflow |
| `throttled=0x4` | Currently throttling — temperature too high |

**Webconf dashboard** shows CPU and temperature live at `http://zynthian.local`.

---

## Engine CPU Cost (Pi 4, approximate)

These figures assume one engine instance at default settings:

| Engine | Notes | Approx CPU |
|--------|-------|------------|
| ZynAddSubFX (simple preset) | Additive + subtractive synthesis | 8–15% |
| ZynAddSubFX (complex: many oscillators, effects) | Full poly preset | 15–30% |
| FluidSynth | SF2 playback, 64 voice poly | 3–8% |
| setBfree | Continuous tonewheel emulation | 5–10% |
| LinuxSampler | Sample playback, disk-bound | 2–5% + disk I/O |
| Sfizz | SFZ player | 3–7% |
| Aeolus | Pipe organ model | 8–15% |
| Pianoteq (if licensed) | Physical model piano | 10–20% |
| AudioPlayer | File playback | 1–3% |
| SooperLooper | Live looper | 3–6% |
| Calf Reverb (LV2) | Simple algorithmic reverb | 2–4% |
| IR.lv2 (convolution reverb) | Per IR length | 10–35% |
| ZynAddSubFX as organ emulation | Many additive oscillators | 15–25% |

---

## Polyphony Limits

Maximum simultaneous voices before CPU saturates:

| Engine | Pi 3 | Pi 4 | Pi 5 |
|--------|------|------|------|
| ZynAddSubFX (typical preset) | 8–16 | 24–48 | 48–96 |
| FluidSynth | 32–64 | 64–128 | 128–256 |
| LinuxSampler | 16–32 | 32–64 | 64–128 |

Set FluidSynth polyphony via env var `ZYNTHIAN_SYNTH_POLYPHONY` in `zynthian_envars.sh` (default 64). Reducing to 32 saves ~30% FluidSynth CPU with minimal audible impact for most music styles.

ZynAddSubFX polyphony is set per-part within the engine's own parameter pages.

---

## CPU Optimization Strategies

### Quick Wins

| Strategy | Typical CPU saving |
|----------|------------------|
| Disable DPM meters (`enable_dpm = False`) | 1–2% |
| Reduce FluidSynth polyphony 64 → 32 | 2–4% |
| Reduce ZynAddSubFX polyphony 64 → 16 | 5–15% depending on preset |
| Use setBfree instead of ZynAddSubFX organ emulation | Saves 5–15% |
| Remove unused chains (don't just mute) | Saves idle engine CPU |
| Replace convolution reverb with algorithmic reverb | Saves 10–30% |
| Pre-render complex pads as audio → use AudioPlayer | Replaces 10–25% with 1–3% |
| Increase JACK buffer 256 → 512 | Eliminates XRUNs at +6ms latency cost |

### Structural Strategies

- **Shared reverb bus:** one Calf Reverb on a mixbus chain, fed by all instrument chains via sends, instead of per-chain reverb. One reverb CPU cost vs N.
- **ZS3 instead of snapshots during performance:** ZS3 recall is instant; full snapshot reload restarts engines (1–5s gap).
- **Limit simultaneous LV2 plugins:** each plugin instance runs its own audio processing thread. 10 LV2 plugins on one chain costs more than 10 sequential processors in terms of scheduling overhead.
- **Reduce sample rate to 44100 Hz:** default is already 44100. Moving to 48000 Hz increases CPU ~9% for no audible benefit for synth use.

---

## Example: Stable 4-Chain Live Setup on Pi 4

Target: no XRUNs at `-p 256` (5.8ms latency), comfortable headroom below 70% CPU.

| Chain | Engine | Preset type | Poly | Est. CPU |
|-------|--------|-------------|------|---------|
| Piano | FluidSynth (Salamander) | Piano | 48 | 6% |
| Organ | setBfree | Full organ | — | 8% |
| Strings pad | ZynAddSubFX | Simple additive pad | 16 | 8% |
| Drums | FluidSynth (GM drums) | Percussion | 24 | 3% |
| Reverb send bus | Calf Reverb LV2 | — | — | 2% |
| **Total** | | | | **~27%** |

JACK settings:
```bash
JACKD_OPTIONS="-P 70 -t 2000 -s a -d alsa -d hw:S2 -r 44100 -p 256 -n 3"
```

---

## Diagnosing XRUNs

Systematic approach:

1. Check CPU% in status bar — is it above 75%?
2. SSH in: `htop` — which process is near 100% on one core?
3. If ZynAddSubFX: reduce polyphony and/or patch complexity.
4. Check temperature: `vcgencmd measure_temp` — above 75°C improve airflow.
5. Check throttle state: `vcgencmd get_throttled` — non-zero means power or thermal issues.
6. Increase JACK buffer: webconf → Audio → Period Size → 512.
7. If still XRUNing: remove the heaviest chain and substitute a lighter engine.

---

## What's Next

- [Audio Setup](audio.html) — configure JACK buffer size in webconf
- [Synth Engines](synth-engines.html) — per-engine CPU notes
- [Troubleshooting](troubleshooting.html) — XRUNs and audio glitches
- [Admin & System](admin-guide.html) — DPM toggle and visible chains settings

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_admin.py`, `zyngui/zynthian_gui_mixer.py`, `zyngui/zynthian_gui_dpm.py`, `zynthian-sys/config/zynthian_envars_V5.sh`.*
