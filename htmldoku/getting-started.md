# Getting Started with Zynthian

This tutorial walks you from a blank SD card to a working Zynthian box producing sound. Each step must succeed before moving to the next.

---

## What You Need

Before starting, gather:

- Raspberry Pi 4 (Pi 3/5 also supported)
- MicroSD card, 16 GB minimum (32 GB recommended; Samsung or SanDisk Endurance class)
- USB-C power supply, 5V / 3A minimum
- HDMI cable + TV or monitor (connect to **HDMI0** — the port closest to USB-C power)
- USB keyboard and mouse (optional but useful for initial setup)
- USB or Bluetooth audio device, or a HAT soundcard

---

## Step 1 — Flash the SD Card

Download the latest ZynthianOS image from `https://os.zynthian.org/zynthianos-last-stable.zip`. Unzip it to get a `.img` file.

Flash the image using [Raspberry Pi Imager](https://www.raspberrypi.com/software/) or [balenaEtcher](https://etcher.balena.io/). Write to the SD card — this takes 5–10 minutes.

> **SD card quality matters.** Budget cards (Intenso, generic) fail under the heavy random-write load of first boot. Use Samsung Endurance Pro or SanDisk Endurance if the default card causes filesystem errors (`EXT4-fs error` in `dmesg`).

---

## Step 2 — First Boot

Insert the SD card and power on the Pi. First boot takes **10–15 minutes** and reboots once automatically. During this time the system:

1. Runs hardware autodetection (i2c scan for DAC, ADC, GPIO expanders)
2. Regenerates SSH/SSL keys (unique per device)
3. Creates a WiFi access point named `Zynthian`
4. Regenerates the LV2 plugin cache
5. Expands the root filesystem to fill the SD card
6. Reboots into the normal UI

Do not power off during first boot.

---

## Step 3 — Access the Device

After the reboot you can reach Zynthian three ways:

**Browser (recommended):** Open `http://zynthian.local` in any browser on the same network. The webconf dashboard appears. Default password: `opensynth`.

**SSH:** `ssh root@zynthian.local` — password `opensynth`. If `.local` doesn't resolve, find the IP from your router or use the WiFi hotspot address `192.168.11.1`.

**WiFi hotspot:** If no cable is connected, Zynthian creates its own WiFi network named `Zynthian`. Connect to it, then open `http://zynthian.local`.

---

## Step 4 — Connect Your Display

Plug HDMI into **HDMI0** (the port closest to the USB-C power connector on Pi 4). Zynthian defaults to 1280×720. If the screen stays black, add HDMI force settings via SSH [`zynthian-sys/sbin/config-on-boot.sh:1-30`]:

```bash
cat >> /boot/firmware/config.txt << 'EOF'
hdmi_force_hotplug=1
hdmi_drive=2
hdmi_group=1
hdmi_mode=4
EOF
reboot
```

For old 720p TVs, `hdmi_mode=4` (720p) works. For very old 480p displays, try `hdmi_mode=1`.

---

## Step 5 — Enable Touch Navigation

Zynthian's V5 touch keypad is a software button panel that overlays the left side of the display. It replaces physical hardware buttons and is required for all tutorial navigation.

In VNC (or directly on the display if it's showing the Zynthian UI), navigate to **Admin → Touch Navigation → V5 keypad at left**.

The display reloads and shows a 160 px button panel on the left side. All tutorials assume this panel is active. See [UI Navigation](ui-navigation.html) for the full button reference.

---

## Step 6 — Configure Audio

Open `http://zynthian.local` → **Hardware** → **Audio**. Select your audio device from the **Soundcard** dropdown.

**USB audio devices** appear by their ALSA name (e.g. `Sound Blaster Play! 2` shows as `S2`). Set sample rate to **44100** for most USB devices — check `cat /proc/asound/card*/stream0` via SSH to confirm what your device supports.

**Built-in headphone jack** is available as `Headphones` but delivers lower quality than a dedicated soundcard.

Click Save, then Reboot.

---

## Step 7 — Verify Audio

After reboot, check JACK is running via SSH:

```bash
systemctl status jack2 --no-pager
```

The status should show `active (running)` with your chosen audio device. If JACK fails, check the troubleshooting page.

---

## Step 8 — Load a Synth and Play

Tap **+** at the right edge of the Mixer screen. The **Add Chain** screen appears — tap **Instrument**, select an engine (start with **ZynAddSubFX** or **FluidSynth**), pick a bank, and select a preset. Connect a USB MIDI keyboard and play.

---

## What's Next

- [Understanding Zynthian](userguide.md) — learn about chains, engines, and snapshots
- [Audio Setup](audio.md) — tune JACK buffer size and latency
- [MIDI Controllers](midi.md) — connect a physical keyboard or pad controller
- [Troubleshooting](troubleshooting.md) — if anything above didn't work

---

*Version: 2026-05-25 — derived from `zynthian-sys/scripts/first_boot.sh` and `zynthian-sys/sbin/config-on-boot.sh`.*
