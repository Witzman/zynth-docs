# Troubleshooting

Common problems and fixes organized by symptom. Check [Architecture](architecture.md) if you need to understand *why* something works the way it does.

---

## No Display / Black Screen

**Problem:** HDMI cable plugged in, Pi powered on, but screen is black or shows no signal.

**Most likely cause:** Wrong HDMI port. Pi 4 has two HDMI ports. Use **HDMI0** (the port closest to the USB-C power connector).

**Fix:** Unplug HDMI and replug into HDMI0 (no reboot needed).

If the correct port is already in use and still no signal:

```bash
# Check /boot/firmware/config.txt (NOT /boot/config.txt on newer Pi OS)
grep -E "hdmi_force|hdmi_drive|hdmi_group|hdmi_mode" /boot/firmware/config.txt
```

For old TVs, add these lines to `/boot/firmware/config.txt`:
```ini
hdmi_force_hotplug=1
hdmi_drive=2
hdmi_group=1
hdmi_mode=4
```

Reboot after editing.

---

## Zynthian UI Shows Error Splash / Red "Error" Screen

**Problem:** The Zynthian UI loads but shows a dude picture and red "error" text. Audio/MIDI counters run in the top bar.

**Cause:** JACK audio server is not running. The UI requires JACK before it can start engines.

**Diagnose:**
```bash
systemctl status jack2 --no-pager
journalctl -u jack2 -n 30 --no-pager
```

**Fix depends on what `journalctl` shows:**

| JACK Log Line | Meaning | Fix |
|---|---|---|
| `ALSA: cannot configure capture channel` | Sample rate mismatch | Set 44100 Hz in webconf → Hardware → Audio |
| `usb_set_interface failed (-32)` | USB device issue | Move USB audio to USB 2.0 (black) port |
| `Cannot open device` | Card name wrong or device missing | Check `aplay -l`; set correct card in webconf |
| `Failed to open server` | Generic JACK start fail | See USB audio section below |

---

## No Sound / JACK Not Starting

### USB Audio Device Fails to Open

**Symptom:** `journalctl -u jack2` shows `usb_set_interface failed (-32)` or `Broken pipe`.

**Fix 1 — Use USB 2.0 port:**
Some USB audio devices fail on USB 3.0 (blue) ports. Move the device to a USB 2.0 (black) port on the Pi.

**Fix 2 — Add skip_validation quirk:**
```bash
# Get vendor:product IDs
lsusb | grep -i audio

# Add quirk (replace VVVV:PPPP with your device)
echo 'options snd-usb-audio vid=0xVVVV pid=0xPPPP skip_validation=y' >> /etc/modprobe.d/usb-audio.conf
rmmod snd-usb-audio && modprobe snd-usb-audio
```

**Fix 3 — Sample rate mismatch:**
Some USB devices only support 44100 Hz. If `aplay -l` shows the device but JACK fails:
1. Open webconf → **Hardware** → **Audio**
2. Set Sample Rate to **44100**
3. Save and reboot

### Verify audio before starting JACK:
```bash
aplay -l                          # list all devices
speaker-test -D hw:S2 -c 2 -r 44100 -t sine   # replace S2 with your card name
```

---

## Audio Card Not Found After Reboot

**Symptom:** `aplay -l` shows a different card number or hex name after reboot.

**Cause:** USB card numbers (`hw:0`, `hw:1`) change between reboots. Zynthian uses the ALSA card name (`hw:S2`) for stability.

**Check current card name:**
```bash
aplay -l | grep -i card
cat /proc/asound/cards
```

If the name changed (e.g. from `S2` to `U0x41e0x323d`), verify the device is in the same USB port and the udev rule is applied. Replug into the same physical USB port and reboot.

---

## Webconf Not Loading

**Symptom:** Browser shows connection refused or timeout at `http://zynthian.local`.

**Check service status:**
```bash
systemctl status zynthian-webconf --no-pager
journalctl -u zynthian-webconf -n 30 --no-pager
```

**Common cause — libzyncore.so missing:**
```
ImportError: libzyncore.so: cannot open shared object file
```

Fix: rebuild the native library:
```bash
cd /zynthian/zyncoder
mkdir -p build && cd build
cmake .. && make -j4
```

Then restart webconf:
```bash
systemctl restart zynthian-webconf
```

**Second common cause — port conflict:**
Check if port 80 is occupied:
```bash
ss -tlnp | grep :80
```

---

## Zynthian Not Reachable by Hostname (zynthian.local)

**Problem:** `ssh root@zynthian.local` or `http://zynthian.local` fails to resolve.

**Cause:** mDNS (Avahi) not working on your network or OS.

**Try by IP address instead:**
```bash
# On the Pi (if you can reach it):
hostname -I

# Or check your router DHCP table for a device named "zynthian"
```

**On Windows:** Install Bonjour or use the IP directly. mDNS requires Bonjour to resolve `.local` names.

**On Linux:** Install `avahi-daemon` if not present.

---

## SD Card Corruption

**Symptom:** Random crashes, filesystem errors in `dmesg`, services fail to start after updates, UI freezes.

**Diagnose:**
```bash
dmesg | grep -E "EXT4|mmcblk|I/O error"
```

If you see `EXT4-fs error: checksumming directory block` or `I/O error on mmcblk0p2`, the SD card is corrupted or failing.

**Fix:** Reflash with a quality SD card. Avoid budget/unbranded cards. Recommended: SanDisk Ultra, Samsung EVO.

> **Warning:** Do not run `raspi-config --expand-rootfs` on poor-quality SD cards. The filesystem expansion during first boot can corrupt the card under write stress.

**Prevention:**
- Use a quality card (Class 10 / A1 rated)
- Check `vcgencmd get_throttled` — undervoltage causes SD corruption:
  ```bash
  vcgencmd get_throttled
  # 0x0 = healthy; 0x50000 = past undervoltage
  ```
- Use a 5V/3A USB-C power supply on Pi 4

---

## MIDI Controller Not Responding

**Controller not detected:**
```bash
dmesg | grep -i "usb\|midi" | tail -10
aconnect -l
```

If device appears in `dmesg` but not `aconnect -l`:
- Try a different USB cable (data cable, not charge-only)
- Some devices need the MIDI class driver: `modprobe snd-usb-midi`

**Notes play but wrong engine responds:**
- Check MIDI channel: chain → options → **MIDI Channel**
- Use MIDI monitor to see what's arriving:
  ```bash
  amidi -p hw:X,0,0 -d   # replace X with card number
  ```

**All notes stuck / no note-off:**
```bash
# Send all-notes-off on all channels
for ch in $(seq 0 15); do amidi -p hw:X,0,0 -S "B${ch}7B00"; done
```

---

## Bluetooth MIDI Pairing Fails

**Error:** `[bluetooth]# pair XX:XX:XX:XX:XX:XX` returns `AuthenticationCanceled`.

**Cause:** Device timed out of pairing mode before the `pair` command completed.

**Fix:** Put the BLE MIDI device into pairing mode *immediately* before running `pair`. Type the `pair` command first (don't press Enter), activate pairing mode on the device, then press Enter within 3 seconds.

```bash
bluetoothctl
power on
scan on
# Wait for device to appear, then:
pair XX:XX:XX:XX:XX:XX   # device must be in pairing mode RIGHT NOW
trust XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
```

---

## Undervoltage / Power Issues

**Symptom:** Lightning bolt icon on screen, random reboots, USB device instability.

```bash
vcgencmd get_throttled
# 0x0 = healthy
# 0x50000 = past undervoltage event
```

Use a **5V / 3A USB-C** supply. Cheap phone chargers often drop below 4.8V under Pi 4 load.

---

## First Boot Never Completes

**Symptom:** Pi has been running for 10+ minutes, webconf still not accessible, no activity.

**Check first_boot service:**
```bash
systemctl status first_boot --no-pager
journalctl -u first_boot -n 50 --no-pager
```

First boot runs: hardware autoconfig → ALSA fix → SSH key regen → WiFi AP creation → LV2 cache rebuild → filesystem expand → reboot. LV2 cache rebuild can take 5–10 minutes on Pi 3.

If service shows `failed`, check the journal for the specific step that failed. Most common cause: SD card write error during filesystem expand.

---

## What's Next

- [Hardware Setup](hardware.md) — display and audio hardware reference
- [Audio Setup](audio.md) — JACK configuration details
- [Architecture](architecture.md) — understand the system components

---

*Version: 2026-05-25 — derived from `zynthian-sys/sbin/`, `zynthian-sys/etc/systemd/`, field experience.*
