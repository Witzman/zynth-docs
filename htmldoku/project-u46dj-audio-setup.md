# ESI U46DJ USB Audio Setup

**Goal:** Configure the ESI U46DJ as Zynthian's audio device at 44.1 kHz (4 in / 6 out), verify output to monitors, and confirm all four inputs are available.
**Prerequisites:** Zynthian booted and accessible via SSH and VNC. Monitors connected to U46DJ Mix Out (rear panel RCA stereo pair). E-MU Xboard connected.
**Access:** SSH · VNC

---

## Part 1 — Configure JACK for U46DJ `[draft]`

The U46DJ has no preset in Zynthian's webconf audio settings. Configure JACK directly by editing the service file via SSH.

### Step 1 — Power on the U46DJ and connect USB

Press the power button on the U46DJ front panel. The power LED lights up.

Connect the USB cable from the U46DJ to a USB port on the Raspberry Pi.

**Verify:** Power LED is lit.

### Step 2 — Find the ALSA device name

```bash
ssh root@zynthian.local
aplay -l | grep -i esi
```

Expected output:
```
card N: U46DJ [U46DJ], device 0: USB Audio [USB Audio]
```

The name `U46DJ` in the second bracket is the ALSA card identifier. The JACK config will reference it as `hw:U46DJ`.

**Verify:** A line containing `U46DJ` appears. If nothing appears, confirm the USB cable is seated and the device is powered on, then re-run.

### Step 3 — Stop Zynthian and JACK

```bash
systemctl stop zynthian
systemctl stop jack2
```

**Verify:** No error output. Both services are stopped.

### Step 4 — Edit the JACK service file

```bash
nano /etc/systemd/system/jack2.service
```

Find the `ExecStart=` line. Replace the entire line with:

```
ExecStart=/usr/bin/jackd -P 70 -s -S -d alsa -d hw:U46DJ -r 44100 -p 256 -n 3 -X raw
```

Save and exit (Ctrl+O, Enter, Ctrl+X).

> **44100 Hz is required.** The U46DJ provides 4 inputs and 6 outputs at 44.1 kHz. At 48 kHz, outputs are limited to 4. The device does not expose a 48 kHz mode in Linux.

> **If you later change audio settings via webconf → Hardware → Audio and save, it will regenerate this file and overwrite the edit.** Re-apply this step if that happens.

**Verify:**
```bash
grep ExecStart /etc/systemd/system/jack2.service
```
Expected: line contains `hw:U46DJ -r 44100`.

### Step 5 — Reload systemd and start JACK

```bash
systemctl daemon-reload
systemctl start jack2
```

Wait 3 seconds, then check:

```bash
systemctl is-active jack2
```

Expected: `active`

If the status is `failed`, check the log:
```bash
journalctl -u jack2 -n 30 --no-pager
```

A common failure is `usb_set_interface failed (-32)`. If that appears, see the Troubleshooting section below.

**Verify:** `jack2` is active.

### Step 6 — Confirm JACK opened the U46DJ

```bash
jack_lsp | grep -v midi
```

Expected:
```
system:capture_1
system:capture_2
system:capture_3
system:capture_4
system:playback_1
system:playback_2
system:playback_3
system:playback_4
system:playback_5
system:playback_6
```

Four capture and six playback ports confirm JACK opened the U46DJ at 44.1 kHz with full 4-in 6-out mode.

**Verify:** Exactly 4 `system:capture_*` and 6 `system:playback_*` ports listed.

### Step 7 — Start Zynthian

```bash
systemctl start zynthian
sleep 5
systemctl is-active zynthian
```

Expected: `active`

**Verify:** Zynthian is running.

### Troubleshooting — JACK fails with `usb_set_interface failed`

If JACK fails, check for this error:
```bash
journalctl -u jack2 -n 50 --no-pager | grep -i 'error\|fail'
```

If you see `usb_set_interface failed (-32)`, add the `skip_validation` quirk for the U46DJ:

```bash
lsusb | grep -i esi
```

Note the `ID XXXX:YYYY` vendor and product values, then:

```bash
echo 'options snd-usb-audio vid=0xXXXX pid=0xYYYY skip_validation=y' >> /etc/modprobe.d/usb-audio.conf
rmmod snd-usb-audio && modprobe snd-usb-audio
systemctl restart jack2
```

Replace `XXXX` and `YYYY` with your actual values from `lsusb`.

---

## Part 2 — Verify Audio Output `[draft]`

Play a synth note and confirm audio reaches the monitors through the U46DJ Mix Out.

### Step 1 — Connect monitors to Mix Out

The **Mix Out** is on the rear panel of the U46DJ — a stereo RCA pair labelled Mix Out. Connect it to powered monitor speakers.

Mix Out carries a hardware mix of all inputs and outputs — it is independent of software volume.

**Verify:** Monitors are connected and powered on.

### Step 2 — Load a synth chain in VNC

Open VNC to `zynthian.local`. In the Zynthian UI:

1. Tap the **+** icon in the main screen.
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

## Part 3 — Verify Inputs `[draft]`

Confirm all four U46DJ inputs are available as JACK capture ports.

### Step 1 — Set the input source selector

The front panel has two physical source selectors:

- **Channel 1/2 selector:** MIC · LINE · PHONO
- **Channel 3/4 selector:** Hi-Z · LINE · PHONO

Set each to match your source:

| Source | Input connector | Selector |
|--------|-----------------|----------|
| Microphone | Channel 1/2 front panel jack (balanced 1/4") | MIC |
| Line-level device | Channel 1/2 or 3/4 rear RCA | LINE |
| Turntable | Rear RCA (Inputs 1–4) | PHONO |
| Guitar or bass | Channel 3/4 front panel jack (unbalanced 1/4") | Hi-Z |

To use a condenser microphone, press **+48V** on the front panel. This enables phantom power on channels 1 and 2 simultaneously.

**Verify:** Selector is at the correct position for your source.

### Step 2 — Connect your source

Connect your instrument or microphone to the corresponding connector.

**Verify:** Cable is connected at both ends.

### Step 3 — Confirm four JACK capture ports

```bash
ssh root@zynthian.local
jack_lsp | grep capture
```

Expected:
```
system:capture_1
system:capture_2
system:capture_3
system:capture_4
```

**Verify:** Exactly four `system:capture_*` ports listed. If fewer appear, JACK opened the device at 48 kHz — confirm the `jack2.service` ExecStart uses `-r 44100` (Part 1 Step 4).

### Step 4 — Route an input to a chain in VNC

Open VNC to `zynthian.local`. In the chain control screen for a loaded ZynAddSubFX chain, look for an audio input routing option and connect `system:capture_1` to the chain's audio input.

Alternatively, connect via SSH:

```bash
jack_lsp -c
```

Look for the chain's input port name in the output. Then:

```bash
jack_connect system:capture_1 <chain-input-port-name>
```

**Verify:** Input signal from your source is audible through the monitors.

---

## Going Further

- Reduce buffer size to `128` frames for lower latency if CPU load is stable; increase to `512` if you hear clicks or dropouts
- Use Output 1–6 (rear RCA) to route separate stems to a hardware mixer instead of the Mix Out
- Connect a turntable to the Phono inputs — the U46DJ includes an RIAA phono preamp built in
- Use the external DC +9V rear power connector when using phantom power to avoid USB current peaks, especially on laptop setups
- Save a Zynthian snapshot after confirming everything works: webconf → **Library → Snapshots**, type a name, click the checkmark
