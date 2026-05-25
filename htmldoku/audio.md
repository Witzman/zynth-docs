# Audio Setup

Zynthian uses JACK as its audio engine. This page covers selecting an audio device, configuring JACK, and tuning latency.

---

## Choosing an Audio Device

Open `http://zynthian.local` → **Hardware** → **Audio**. The **Soundcard** dropdown lists all detected audio devices. Select your device and click Save, then Reboot.

Available device types:

| Type | Notes |
|------|-------|
| **USB Audio** | Plug-and-play. Appears by ALSA name. Sample rate must match device capability. |
| **HifiBerry DAC+** | I2S HAT, high quality. Detected automatically by `zynthian_autoconfig.py`. |
| **ZynADAC** | Official Zynthian audio HAT — best quality option. |
| **bcm2835 Headphones** | Built-in 3.5mm jack. Works but lower quality and higher noise than a dedicated device. |

---

## JACK Configuration

JACK parameters are set in webconf → **Hardware** → **Audio** and written to the environment [`zynthian-sys/config/zynthian_envars_V5.sh`]:

| Parameter | Recommended | Effect |
|-----------|-------------|--------|
| **Sample rate** | 44100 (USB) / 48000 (HAT) | Must match device capability |
| **Buffer size** | 256 | Lower = less latency; higher = more stable |
| **Periods** | 3 | Number of JACK periods per cycle |

JACK starts as `jack2.service` [`zynthian-sys/etc/systemd/jack2.service`] and must be running before the Zynthian UI starts.

---

## Checking JACK Status

```bash
ssh root@zynthian.local
systemctl status jack2 --no-pager
journalctl -u jack2 -n 20 --no-pager
```

A healthy JACK log shows:
```
configuring for 44100Hz, period = 512 frames
ALSA: use 3 periods for capture
ALSA: use 3 periods for playback
```

If JACK fails to start, the Zynthian UI shows an error splash screen and no sound is possible.

---

## USB Audio Device Name Stability

USB audio card numbers (`hw:0`, `hw:1` etc.) change between reboots if devices are plugged in a different order. Zynthian identifies cards by name (`hw:S2`, `hw:U0x41e0x323d`) to avoid this problem [`zynthian-sys/sbin/fix_soundcard_mixer_ctrls.py`].

If a USB device enumerates with an unstable name (long hex string like `U0x41e0x323d`), create a udev rule or use the `snd-usb-audio` quirk to assign a stable name. This happens automatically after the device is selected in webconf and the system is rebooted.

---

## USB Audio Broken Pipe / Interface Errors

Some USB audio devices fail with `usb_set_interface failed (-32)` or `ALSA: cannot configure capture channel`. These are known Linux USB audio issues with specific devices.

Fix: add the `skip_validation` quirk for your device:

```bash
echo 'options snd-usb-audio vid=0xVVVV pid=0xPPPP skip_validation=y' >> /etc/modprobe.d/usb-audio.conf
rmmod snd-usb-audio && modprobe snd-usb-audio
```

Replace `VVVV` and `PPPP` with your device's vendor and product IDs from `lsusb`. Confirm the fix with `speaker-test -D hw:DeviceName -c 2 -r 44100 -t sine`.

---

## Latency Tuning

Lower buffer size reduces latency but increases CPU load and XRUN risk. Start at 512 and reduce if the system is stable:

- **512 frames at 44100 Hz** ≈ 11.6ms round-trip (good for most use)
- **256 frames at 44100 Hz** ≈ 5.8ms (low latency; needs stable USB/HAT device)
- **128 frames** — only stable with I2S HATs on Pi 4

Monitor XRUNs in the JACK log:
```bash
journalctl -u jack2 -f | grep -i xrun
```

---

## What's Next

- [Getting Started](getting-started.md) — initial audio device selection
- [Troubleshooting](troubleshooting.md) — JACK errors and no-sound situations
- [Hardware Setup](hardware.md) — I2S HAT installation

---

*Version: 2026-05-25 — derived from `zynthian-sys/etc/systemd/jack2.service`, `zynthian-webconf/lib/audio_config_handler.py`.*
