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
| ESI U46DJ | USB | `U46DJ` [low] |
| USB Sound Blaster Play! 2 | USB | `S2` (after udev rule) |
| Generic USB audio | USB | Varies; check with `aplay -l` |
| bcm2835 headphones | Built-in | `Headphones` |

For USB audio devices with `usb_set_interface failed` errors, add a `snd-usb-audio` quirk — see [Audio Setup](audio.md).

### ESI U46DJ

4-in 6-out USB audio interface by ESI. USB Audio Class 1.0 — class-compliant, no driver needed on Linux.

**I/O summary:**

| | Connectors | Notes |
|-|------------|-------|
| Inputs | 4× RCA (rear) | Shared Line/Phono per pair. Phono has RIAA preamp for moving-magnet cartridges. |
| Mic in | 1/4" balanced (front, CH 1/2) | +48V phantom power via front button. MIC/LINE/PHONO selector. |
| Hi-Z in | 1/4" unbalanced (front, CH 3/4) | Guitar/instrument direct. Hi-Z/LINE/PHONO selector. |
| Outputs | 6× RCA (rear) | Line level, -10dBV nominal / +6dBV max. |
| Mix out | 2× RCA (rear) | Stereo monitor mix output — connect to powered monitors. |
| Headphone | 1/4" (front) | Volume control on front panel. 125mW max @ 32 ohm. |
| Ground lug | Screw terminal (rear) | Turntable ground — connect when using phono inputs to prevent hum. |

**Sample rate constraint (USB 1.1 bandwidth):**

| Rate | Inputs | Outputs |
|------|--------|---------|
| 44.1 kHz | 4 | 6 |
| 48 kHz | 4 | 4 (outputs 5/6 inactive) |

Zynthian defaults to 48 kHz — at this rate only 4 outputs are available. Switch to 44.1 kHz in webconf → **Hardware → Audio** if all 6 outputs are needed.

**Phantom power warning:** When using a condenser microphone, the USB bus may not supply enough current. Use an external DC 9V/500mA adapter via the rear power connector if the device becomes unstable when phantom power is active.

Verify detection after connecting via USB:

```bash
aplay -l
# → card X: U46DJ [U46DJ], device 0: ...
```

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

## MIDI Controllers

| Device | Type | Connection | Notes |
|--------|------|------------|-------|
| E-MU Xboard (25/49/61) | Keyboard controller | USB or MIDI DIN | 16 velocity-sensitive keys (25/49/61 key), aftertouch, pitch/mod wheels, 16 assignable CC knobs, 16 patches, 4 keyboard zones. Powers via USB, 6VDC adapter, or 3× AA batteries. Can act as USB-MIDI interface for a DAW. |
| SMC-PAD | Pad controller | USB-C or Bluetooth 5.0 | 16 RGB velocity+aftertouch pads, 8 assignable 360° encoders, DAW transport (Play/Stop/Record), track navigation, note repeat with swing/sync/tap tempo. Also has 3.5mm MIDI OUT. Built-in 2000mAh battery. Requires BLE MIDI driver on Windows; natively supported on Mac/iOS/Android. |

For MIDI routing and mapping these controllers in Zynthian, see [MIDI Setup](midi.md).

---

## What's Next

- [Getting Started](getting-started.md) — initial hardware connection
- [Audio Setup](audio.md) — audio device configuration
- [Troubleshooting](troubleshooting.md) — display and power issues

---

*Version: 2026-05-25 — derived from `zynthian-sys/sbin/zynthian_autoconfig.py`, `zynthian-sys/config/wiring-profiles/`, `zynthian-webconf/lib/hwoptions_config_handler.py`.*
