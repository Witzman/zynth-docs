# Performance & CPU Management

Running multiple synth engines simultaneously on a Raspberry Pi requires attention to CPU budget. This page explains how to read the performance indicators and tune your setup for stability.

---

## Status Bar CPU Indicator

The status bar at the top of every screen shows a CPU percentage. This is the total system CPU load across all cores.

| Range | Meaning | Action |
|-------|---------|--------|
| 0–50% | Comfortable | No action needed |
| 50–75% | Moderate | Monitor, avoid adding heavy engines |
| 75–85% | High | Risk of XRUNs under heavy MIDI load |
| 85–100% | Critical | Expect audio glitches; reduce load |

On Pi 4 (4 cores), CPU% reflects the sum across all 4 cores divided by 4. A 4-chain setup can saturate one core at 25% total — misleading. Watch individual JACK client CPU in `htop` if you suspect one engine is a bottleneck.

---

## DPM — Digital Peak Meters

The mixer shows vertical level meters (DPM) for each chain when enabled. These are **pre-clipping indicators** — the bar should not reach the top red zone.

| Meter zone | Meaning |
|------------|---------|
| Green (bottom 2/3) | Safe level |
| Yellow (upper third) | Approaching 0 dBFS |
| Red (top) | Clipping — reduce volume |

**Enable/disable DPM:** Admin → AUDIO → **Mixer Peak Meters** (toggles `enable_dpm`). Disabling DPM saves a small amount of CPU.

---

## JACK XRUN Indicator

An **XRUN** (under-run or over-run) occurs when the CPU cannot process an audio buffer in time. The result is an audible click or dropout.

The status bar shows an XRUN counter (often a small ✕ icon with a count). If this increments during performance, the CPU cannot keep up.

**Immediate response to XRUNs:**

1. Check CPU% — is it above 80%?
2. Check temperature: `ssh root@zynthian.local "vcgencmd measure_temp"` — above 80°C throttles the Pi.
3. Reduce polyphony on the heaviest engine (ZynAddSubFX part settings).
4. Increase JACK buffer size (webconf → Audio → Period Size).

**JACK config** (from `zynthian_envars.sh`):
```bash
JACKD_OPTIONS="-P 70 -t 2000 -s a -d alsa -d hw:S2 -r 44100 -p 256 -n 3"
```
- `-p 256` = 256 frames per period ≈ 5.8ms latency at 44100 Hz
- `-p 512` = 512 frames ≈ 11.6ms — use if XRUNs persist

---

## Engine CPU Cost (Pi 4, approximate)

| Engine | Polyphony | Approximate CPU per chain |
|--------|-----------|--------------------------|
| ZynAddSubFX | 8–64 voices | 8–25% |
| FluidSynth | 64–256 voices | 3–8% |
| setBfree | Continuous | 5–10% |
| LinuxSampler | 64 voices | 2–5% (disk I/O bound) |
| Sfizz | 64 voices | 3–7% |
| Aeolus | Continuous | 8–15% |
| LV2 (simple plugin) | N/A | 1–3% |
| LV2 (convolution reverb) | N/A | 10–30% |
| Audio Player | N/A | 1–3% |
| SooperLooper | N/A | 3–6% |

These are rough estimates. Actual cost depends on preset complexity (ZynAddSubFX varies enormously), sample file size (LinuxSampler), and IR length (convolution reverbs).

---

## Polyphony Limits

| Pi Model | Safe simultaneous voices (ZynAddSubFX) |
|----------|---------------------------------------|
| Pi 3 | 16–24 |
| Pi 4 | 32–64 |
| Pi 5 | 64–128 |

For FluidSynth, set `ZYNTHIAN_SYNTH_POLYPHONY` in `zynthian_envars.sh` (default 64). Reducing to 32 saves CPU with minimal audible difference in most setups.

---

## Temperature Monitoring

Raspberry Pi 4 throttles at 80°C and hard-throttles at 85°C, causing XRUNs and sluggish UI.

**Check temperature:**
```bash
ssh root@zynthian.local "vcgencmd measure_temp"
# → temp=55.2'C
```

**Check throttle state:**
```bash
ssh root@zynthian.local "vcgencmd get_throttled"
# → throttled=0x0   (OK)
# → throttled=0x50000  (was throttled, now OK)
# → throttled=0x5      (currently throttled!)
```

The Zynthian case with heatsink keeps the Pi 4 below 65°C in normal use. Convolution reverbs or many simultaneous LV2 chains can push it higher.

**Webconf dashboard** shows CPU and temperature at `http://zynthian.local` → Dashboard tab.

---

## Optimization Strategies

**Quick wins:**

| Strategy | Savings |
|----------|---------|
| Disable DPM meters | ~2% |
| Reduce ZynAddSubFX polyphony from 64 to 32 | 5–15% |
| Use Audio Player instead of live ZynAddSubFX for pads | 10–20% |
| Increase JACK buffer from 256 to 512 | Eliminates XRUNs at cost of +6ms latency |
| Disable unused chains (mute vs stop) | Mute saves nothing; remove unused chains entirely |
| Use setBfree instead of ZynAddSubFX organ | setBfree ~5%, ZynAddSubFX organ ~15% |

**Structural approaches:**

- Pre-render complex pads as audio files, play them back via Audio Player.
- Use LV2 reverb/delay only on the main mixbus (one reverb shared by all chains) instead of per-chain reverb.
- On Pi 4, limit to 3–4 simultaneous synth engines plus effects.

---

## Example: Stable 4-Chain Live Setup on Pi 4 at Buffer 256

| Chain | Engine | Polyphony | CPU est. |
|-------|--------|-----------|---------|
| Piano | FluidSynth (Steinway) | 64 | 5% |
| Organ | setBfree | continuous | 7% |
| Lead | ZynAddSubFX (Simple Lead) | 16 | 8% |
| Drums | FluidSynth GM drums | 32 | 3% |
| Reverb bus | Calf Reverb LV2 | — | 2% |

Total: ~25% CPU, comfortable headroom for MIDI processing and UI.

```bash
JACKD_OPTIONS="-P 70 -t 2000 -s a -d alsa -d hw:S2 -r 44100 -p 256 -n 3"
```

---

## What's Next

- [Audio Setup](audio.html) — configure JACK buffer size in webconf
- [Synth Engines](synth-engines.html) — per-engine CPU notes
- [Troubleshooting](troubleshooting.html) — diagnosing XRUNs and glitches

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_admin.py`, `zyngui/zynthian_gui_mixer.py`, `zynthian-sys/config/zynthian_envars_V5.sh`.*
