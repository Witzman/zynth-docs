# ESI U46DJ USB Audio Setup

**Goal:** Configure the ESI U46DJ as Zynthian's audio device at 44.1 kHz (4 in / 6 out), verify output to monitors, and confirm all four inputs are available.
**Prerequisites:** Zynthian booted and accessible via SSH and webconf. E-MU Xboard connected and working. Monitors connected to U46DJ Mix Out.
**Access:** SSH · Webconf · VNC

---

## Part 1 — Select Device and Verify JACK `[draft]`

Select the U46DJ in webconf and confirm JACK starts with it.

### Step 1 — Power on the U46DJ

Press the power button on the front panel of the U46DJ. The power LED on the front panel lights up.

**Verify:** Power LED is lit.

### Step 2 — Connect USB to the Pi

Connect the USB cable from the U46DJ to a USB port on the Raspberry Pi.

**Verify:** No action needed — Linux detects it silently.

### Step 3 — Open the Audio settings in webconf

Open `http://zynthian.local` in a browser. Go to **Hardware → Audio**.

**Verify:** The Audio configuration page loads and shows the **Soundcard** dropdown.

### Step 4 — Select the U46DJ

In the **Soundcard** dropdown, select the entry that contains **U46DJ**. It may appear as `U46DJ` or with a longer ALSA identifier such as `U0x46d0x...`.

If U46DJ does not appear in the list:
- Confirm USB cable is connected and device is powered on.
- Reload the page — webconf rescans detected devices on load.

**Verify:** U46DJ is selected in the Soundcard dropdown.

### Step 5 — Set sample rate and JACK parameters

Set the following fields:

| Field | Value |
|-------|-------|
| **Sample Rate** | `44100` |
| **Buffer Size** | `256` |
| **Periods** | `3` |

44.1 kHz gives you 4 inputs and 6 outputs. 48 kHz would reduce outputs to 4.

**Verify:** Fields show 44100 / 256 / 3.

### Step 6 — Save and reboot

Click **Save**. When prompted, click **Reboot**.

Wait 30–60 seconds for the Pi to restart, then reload `http://zynthian.local`.

**Verify:** webconf loads after reboot.

### Step 7 — Confirm JACK is running

```bash
ssh root@zynthian.local
systemctl status jack2 --no-pager
```

Expected:
```
Active: active (running)
```

Then check the JACK log:
```bash
journalctl -u jack2 -n 30 --no-pager
```

Expected lines:
```
configuring for 44100Hz, period = 256 frames
ALSA: use 3 periods for capture
ALSA: use 3 periods for playback
```

If JACK shows `failed` or `inactive`, see the Troubleshooting section below.

**Verify:** JACK is `active (running)` and log shows `44100Hz`.

### Troubleshooting — JACK fails to start

If JACK fails, check the full error:
```bash
journalctl -u jack2 -n 50 --no-pager | grep -i error
```

Common cause — `usb_set_interface failed (-32)`: add the skip_validation quirk. First find your device's USB IDs:
```bash
lsusb | grep -i ESI
```

Note the `ID xxxx:yyyy` values, then:
```bash
echo 'options snd-usb-audio vid=0xXXXX pid=0xYYYY skip_validation=y' >> /etc/modprobe.d/usb-audio.conf
rmmod snd-usb-audio && modprobe snd-usb-audio
systemctl restart jack2
```

Replace `XXXX` and `YYYY` with your actual vendor and product IDs.

---

## Part 2 — Verify Audio Output `[draft]`

Play a synth note and confirm audio reaches monitors through the Mix Out.

### Step 1 — Connect monitors to Mix Out

The **Mix Out** (RCA stereo pair) is on the rear panel of the U46DJ. Connect it to powered monitor speakers.

Mix Out carries a hardware mix of all outputs — it is independent of software volume. Turn monitor volume to a comfortable level.

**Verify:** Monitors are connected and powered on.

### Step 2 — Load a synth chain in VNC

Open VNC to `zynthian.local`. In the Zynthian UI:

1. Tap the **+** icon (bottom-left of screen) to add a new chain.
2. Tap **Instrument**.
3. Select **ZynAddSubFX** as the engine.
4. Choose any preset from the list (the default preset works).

**Verify:** A chain appears in the main screen with ZynAddSubFX loaded.

### Step 3 — Play a note

On the E-MU Xboard keyboard, press any key.

**Verify:** Sound comes from the monitors. If no sound: confirm the JACK log showed the U46DJ device name (not `bcm2835` or another device), and that the E-MU Xboard MIDI port is active in webconf → **MIDI → Ports**.

---

## Part 3 — Verify Inputs `[draft]`

Confirm that all four U46DJ inputs are available as JACK capture ports, and that your source reaches Zynthian.

### Step 1 — Set the input source selector on the U46DJ

The front panel has two physical source selectors:

- **Channel 1/2 selector:** MIC · LINE · PHONO
- **Channel 3/4 selector:** Hi-Z · LINE · PHONO

Set each selector to match your source. Options:

| Source | Input | Selector |
|--------|-------|----------|
| Microphone | Channel 1/2 front panel jack (balanced 1/4") | MIC |
| Line-level device | Channel 1/2 or 3/4 rear RCA | LINE |
| Turntable | Rear RCA (Inputs 1–4) | PHONO |
| Guitar / bass | Channel 3/4 front panel jack (unbalanced 1/4") | Hi-Z |

If using a condenser microphone: press the **+48V** button on the front panel to enable phantom power. This powers both Channel 1 and 2 simultaneously.

**Verify:** Selector is set to the correct position for your source.

### Step 2 — Connect your source

Connect your instrument or microphone to the corresponding connector as shown in the table above.

**Verify:** Cable is connected at both ends.

### Step 3 — Confirm JACK capture ports via SSH

```bash
ssh root@zynthian.local
jack_lsp | grep capture
```

Expected output:
```
system:capture_1
system:capture_2
system:capture_3
system:capture_4
```

Four capture ports confirm JACK opened the U46DJ at 44.1 kHz with full 4-input mode.

**Verify:** Exactly four `system:capture_*` ports are listed.

### Step 4 — Route an input to a chain in VNC

In the Zynthian VNC UI:

1. In the chain you created in Part 2, tap the chain name to open its options.
2. Look for an **Audio Input** or **JACK connections** option.
3. Connect `system:capture_1` and `system:capture_2` to the chain's audio inputs.

Alternatively, use the JACK patchbay via SSH:
```bash
jack_connect system:capture_1 <chain-input-port>
jack_lsp -c
```

Replace `<chain-input-port>` with the actual port name shown by `jack_lsp`.

**Verify:** Input signal from your source is audible through the monitors or visible as activity in the Zynthian UI.

---

## Going Further

- Tune buffer size down to `128` if CPU load is low and you want lower latency
- Use Output 1–6 (rear RCA) to route separate stems to a mixer rather than using Mix Out
- Connect a turntable to the Phono inputs — the U46DJ includes an RIAA phono preamp
- Save a snapshot after confirming everything works (webconf → **Snapshots**)
