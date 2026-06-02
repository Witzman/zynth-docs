# ESI U46DJ USB Audio Setup

**Goal:** Configure the ESI U46DJ as Zynthian's audio device at 44.1 kHz (6-out playback), verify output to monitors via Mix Out.
**Prerequisites:** Zynthian booted and accessible via SSH and touchscreen. U46DJ connected via USB and powered on. Monitors connected to U46DJ Mix Out (rear panel RCA stereo pair). E-MU Xboard connected.
**Access:** SSH · Touchscreen

> **Power supply:** The U46DJ draws up to 500 mA peak at startup — enough to trip the Pi 4's USB port over-current protection. Connect the external DC +9V/500 mA adapter to the rear panel before plugging in the USB data cable. If no DC adapter is available, use a powered USB hub between the Pi and the U46DJ.

> **Boot sequence:** With a powered USB hub, the U46DJ can be left connected and powered on at boot — the hub isolates it from the Pi's USB controller and the JACK wait script handles startup sequencing automatically. Without a powered hub, boot the Pi first, then power on the U46DJ, then connect the USB cable.

> **Capture inputs:** The U46DJ USB capture stream is not stable on this Pi. JACK runs in playback-only mode. The 4 inputs (mic, line, phono, Hi-Z) are available at the hardware level but not routed through JACK in this configuration.

---

## Part 1 — Configure JACK for U46DJ `[verified]`

The U46DJ has no preset in Zynthian's webconf audio settings. Configure JACK directly by editing the service file via SSH, then connect the device using the correct boot sequence.

### Step 1 — Connect U46DJ via powered USB hub and power on

Connect the U46DJ to a powered USB hub, and connect the hub to the Pi. Press the power button on the U46DJ front panel. The power LED lights on.

The powered hub supplies its own 5V — the U46DJ does not draw from the Pi's USB port. The DC +9V rear connector is not required when using a powered hub.

If you have no powered hub: boot the Pi first without U46DJ connected, wait for the Zynthian UI or error screen, then power on the U46DJ and plug in the USB cable.

**Verify:** Power LED is lit. USB cable connected to hub.

### Step 3 — Confirm the U46DJ is detected

```bash
ssh root@zynthian.local
aplay -l | grep -i u46
```

Expected output:
```
card N: U46DJ [U46DJ], device 0: USB Audio [USB Audio]
```

**Verify:** A line containing `U46DJ` appears. If nothing appears and the power LED is lit, try a different USB port on the Pi. If enumeration still fails, see Troubleshooting below.

### Step 4 — Stop Zynthian and JACK

```bash
systemctl stop zynthian
systemctl stop jack2
```

**Verify:** No error output. Both services are stopped.

### Step 5 — Edit the JACK service file

```bash
nano /etc/systemd/system/jack2.service
```

The file must contain these two `Exec` lines in the `[Service]` section. Add or replace as needed:

```
ExecStartPre=/usr/local/bin/wait-for-u46dj.sh
ExecStart=/usr/bin/jackd -P 70 -s -S -d alsa -P hw:U46DJ -r 44100 -p 2048 -n 3 -X raw
```

Save and exit (Ctrl+O, Enter, Ctrl+X).

Notes:
- `-P hw:U46DJ` (capital P) = playback-only. Full duplex (`-d hw:U46DJ`) causes JACK to block on the broken capture stream.
- `-p 2048 -n 3` = 2048-frame buffer, 3 periods (≈139 ms). Smaller buffers cause constant XRUNs on this USB 1.1 device.
- `44100 Hz` is required for 6 outputs. At 48 kHz the device exposes only 4 outputs.
- `ExecStartPre` runs a wait script that holds JACK startup until `U46DJ` appears in ALSA, preventing a race between boot and USB enumeration.

> **If webconf → Hardware → Audio is saved, it will regenerate this file and overwrite the edit.** Re-apply this step if that happens.

**Verify:**
```bash
grep ExecStart /etc/systemd/system/jack2.service
```
Expected: two lines — one with `wait-for-u46dj.sh`, one with `hw:U46DJ -r 44100`.

### Step 6 — Reload systemd and start JACK

```bash
systemctl daemon-reload
systemctl start jack2
```

The wait script runs first — JACK takes about 10 seconds to start. Then check:

```bash
systemctl is-active jack2
```

Expected: `active`

If `failed`, check:
```bash
journalctl -u jack2 -n 30 --no-pager
```

See Troubleshooting below for common failures.

**Verify:** `jack2` is active.

### Step 7 — Confirm JACK opened the U46DJ

```bash
jack_lsp | grep -v midi
```

Expected:
```
system:playback_1
system:playback_2
system:playback_3
system:playback_4
system:playback_5
system:playback_6
```

Six playback ports confirm JACK opened the U46DJ at 44.1 kHz in playback-only mode.

**Verify:** Six `system:playback_*` ports listed.

### Step 8 — Start Zynthian

```bash
systemctl start zynthian
sleep 8
systemctl is-active zynthian
```

Expected: `active`

**Verify:** Zynthian is running and the UI is visible on screen.

---

### Troubleshooting — Device not detected (`aplay -l` shows nothing)

**Firmware stuck state.** The U46DJ USB firmware can get stuck after failed enumeration attempts. Fix:

1. Press the **power button** on the U46DJ front panel — LED off.
2. Wait 5 seconds.
3. Press the power button again — LED on.
4. Re-run `aplay -l | grep -i u46`.

If still not detected, try a different USB port on the Pi (port `1-1.4` may have a corrupted xHCI slot state from over-current events — use a different physical port).

---

### Troubleshooting — JACK active but no playback ports / Zynthian fails to connect

Stale JACK shared memory from a previous session can block client connections. Clear it:

```bash
systemctl stop zynthian
systemctl stop jack2
rm -rf /dev/shm/jack* /dev/shm/jack_db-0
systemctl start jack2
sleep 8
systemctl start zynthian
```

---

### Troubleshooting — JACK reports `Driver is not running`

This means the RT audio thread could not start. Either:

1. The U46DJ capture stream is being opened (full-duplex mode) — fix by ensuring `-P hw:U46DJ` (not `-d hw:U46DJ`) in the `ExecStart` line.
2. A stale jackd process from a previous manual run is holding the device — run `kill -9 $(pgrep jackd)` before starting the service.

---

## Part 2 — Verify Audio Output `[verified]`

Play a synth note and confirm audio reaches the monitors through the U46DJ Mix Out.

### Step 1 — Connect monitors to Mix Out

The **Mix Out** is on the rear panel of the U46DJ — a stereo RCA pair labelled Mix Out. Connect it to powered monitor speakers.

Mix Out carries a hardware mix of all inputs and outputs — it is independent of software volume.

**Verify:** Monitors are connected and powered on.

### Step 2 — Load a synth chain

On the touchscreen:

1. Tap **+** in the mixer screen.
2. Tap **Instrument**.
3. Select **ZynAddSubFX** as the engine.
4. Choose any preset.

**Verify:** A chain appears with ZynAddSubFX loaded.

### Step 3 — Play a note

On the E-MU Xboard, press any key.

**Verify:** Sound comes from the monitors.

If no sound:
- Confirm monitors are on the rear **Mix Out** RCA pair — not Output 1–6.
- Confirm the E-MU Xboard MIDI port is enabled in webconf → **Interface → MIDI Options** → **MIDI Devices**.
- Run `jack_lsp -c | grep 'playback_1'` to confirm audio is routed to `system:playback_1`.

---

## Part 3 — Inputs (not available in current configuration)

The U46DJ USB capture stream (`system:capture_*` ports) is not stable on this Pi — `arecord` returns `Input/output error` when reading from the device. JACK runs in playback-only mode to avoid blocking on the broken capture stream.

The 4 hardware inputs (mic, line, phono, Hi-Z on front and rear panels) are present at the hardware level and accessible via the **Mix Out** monitoring path, but cannot be routed into Zynthian chains in the current configuration.

To use the inputs with Zynthian (e.g. for the Audio FX Chain tutorial), full-duplex mode must first be confirmed working. Change `-P hw:U46DJ` to `-d hw:U46DJ` in the `ExecStart` line and verify that `jack_lsp` shows both `system:capture_*` and `system:playback_*` ports and that JACK remains stable. If JACK blocks and clients cannot connect, revert to playback-only.

---

## Going Further

- Reduce buffer size from `2048` only if the system is fully stable — XRUNs appear quickly on this USB 1.1 device; `2048` is the proven minimum
- Use Output 1–6 (rear RCA) to route separate stems to a hardware mixer instead of the Mix Out
- Connect a turntable to the Phono inputs — the U46DJ includes an RIAA phono preamp built in
- Save a Zynthian snapshot after confirming everything works: webconf → **Library → Snapshots**, type a name, click the checkmark
