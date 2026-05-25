# Hardware Setup

Zynthian runs on Raspberry Pi with a variety of display, audio, and control configurations. This page describes supported hardware and how to configure it.

---

## Supported Raspberry Pi Models

| Model | Support | Notes |
|-------|---------|-------|
| Pi 4 (2/4/8 GB) | Full | Recommended. Strong CPU for multiple synth chains. |
| Pi 5 | Full | Supported; use the V5_pi5 env config. |
| Pi 3B+ | Good | Lighter on CPU; fewer simultaneous engines. |
| Pi Zero 2W | Limited | Possible but constrained by CPU/RAM. |

---

## Official Hardware Kits

Zynthian sells complete kits ([shop.zynthian.org](https://shop.zynthian.org)):

**V5 Kit:** Raspberry Pi 4 + ZynADAC audio codec (PCM5242/PCM1863) + TPA6130 headphone amp + 3.5" touchscreen + 4 rotary encoders + MIDI DIN ports. Detected automatically by `zynthian_autoconfig.py` scanning i2c addresses [`zynthian-sys/sbin/zynthian_autoconfig.py:19-35`].

**Z2 Kit:** Raspberry Pi 4 + ZynADAC + touch display. Similar configuration to V5 with a different board revision.

---

## DIY / Custom Builds

Any Pi with a compatible audio device and a display works. Minimum requirements:

- Raspberry Pi 3/4/5
- Audio: USB audio device or I2S HAT (HifiBerry DAC+, ZynADAC)
- Display: HDMI monitor, official Zynthian screen, or no display (headless with webconf only)

Set the wiring layout in webconf → **Hardware** → **Wiring** to `TOUCH_ONLY` if you have no physical encoders.

---

## Display Options

| Type | Configuration |
|------|---------------|
| HDMI TV / Monitor | Set wiring to `TOUCH_ONLY`; configure `hdmi_force_hotplug=1` in `/boot/firmware/config.txt` if not auto-detected |
| ZynScreen 3.5 | Select in webconf → Display |
| PiScreen 3.5 | Select in webconf → Display |
| Waveshare (various) | Select matching model in webconf → Display |
| No display (headless) | Use webconf only; set wiring to `TOUCH_ONLY` |

For HDMI on old TVs: add `hdmi_force_hotplug=1`, `hdmi_drive=2`, `hdmi_group=1`, `hdmi_mode=4` to `/boot/firmware/config.txt`. Connect to **HDMI0** (closest to USB-C on Pi 4).

---

## Audio Hardware

| Device | Type | ALSA Name |
|--------|------|-----------|
| ZynADAC (PCM5242+PCM1863) | I2S HAT | Detected by autoconfig |
| HifiBerry DAC+ | I2S HAT | `HifiBerry` |
| USB Sound Blaster Play! 2 | USB | `S2` (after udev rule) |
| Generic USB audio | USB | Varies; check with `aplay -l` |
| bcm2835 headphones | Built-in | `Headphones` |

For USB audio devices with `usb_set_interface failed` errors, add a `snd-usb-audio` quirk — see [Audio Setup](audio.md).

---

## Hardware Autodetection

On first boot, `zynthian_autoconfig.py` scans the i2c bus for known chips and selects the matching hardware profile [`zynthian-sys/sbin/zynthian_autoconfig.py:42-120`]:

| i2c Chips Found | Board |
|-----------------|-------|
| PCM1863@0x4A + PCM5242@0x4D + RV3028@0x52 + TPA6130@0x60 + MCP23017@0x20/0x21 | V5 |
| PCM1863@0x4A + PCM5242@0x4D + RV3028@0x52 + MCP23017@0x20/0x21 | Z2 |
| PCM5242@0x4D | HifiBerry DAC+ |

If autodetection selects the wrong profile, override in webconf → **Hardware** → **Wiring**.

---

## Power Supply

The Pi 4 requires a **5V / 3A USB-C** supply. Insufficient power causes:
- Undervoltage (lightning bolt icon on screen)
- Random SD card corruption
- USB device instability

Check throttle status via SSH:
```bash
vcgencmd get_throttled
# 0x0 = healthy; 0x50000 = past undervoltage event
```

---

## What's Next

- [Getting Started](getting-started.md) — initial hardware connection
- [Audio Setup](audio.md) — audio device configuration
- [Troubleshooting](troubleshooting.md) — display and power issues

---

*Version: 2026-05-25 — derived from `zynthian-sys/sbin/zynthian_autoconfig.py`, `zynthian-sys/config/wiring-profiles/`, `zynthian-webconf/lib/hwoptions_config_handler.py`.*
