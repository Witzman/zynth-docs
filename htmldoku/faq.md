# FAQ

Quick answers to common questions. For deeper diagnosis, see [Troubleshooting](troubleshooting.md).

---

## Sound

**Why do I hear no sound?**
JACK is not running. Check `systemctl status jack2` via SSH. Most common causes: wrong sample rate (set 44100 Hz for USB audio in webconf → Hardware → Audio), USB device plugged into a USB 3.0 (blue) port (move it to USB 2.0 black port), or wrong audio device selected in webconf. See [Audio Setup](audio.md).

**Audio works but cuts out with clicks/pops.**
JACK XRUNs — buffer underrun caused by CPU overload or too-small buffer. Increase buffer size from 256 to 512 in webconf → Hardware → Audio. Check `journalctl -u jack2 -f | grep -i xrun`.

**Volume is very low.**
Check the ALSA mixer: `alsamixer -c S2` (replace S2 with your card name). Some USB devices default to 50% gain. Also check webconf → Hardware → Audio Options for input/output gain settings.

**I hear hum / background noise.**
Using built-in headphone jack (`bcm2835 Headphones`) — it has higher noise than any external audio device. Use a USB audio device or I2S HAT for clean audio.

**Sound is distorted at high volumes.**
Input or output gain too high in ALSA mixer. Lower the gain: `alsamixer -c S2`. Also check engine volume in the Zynthian mixer (Main screen → Mixer).

---

## MIDI

**My USB MIDI controller is not responding.**
Check `aconnect -l` via SSH — does the device appear? If not, check `dmesg | grep -i midi | tail -10`. Try a different USB cable (charge-only cables have no data lines). Verify MIDI channel: tap the chain → options → MIDI Channel. Use `amidi -p hw:X,0,0 -d` to see raw MIDI data.

**Controller shows in `aconnect` but notes don't trigger.**
MIDI channel mismatch. Chain defaults to channel 1. Check what channel your controller sends on (usually shown in the controller's settings). Set the chain to **Omni** to respond to all channels temporarily.

**MIDI controller worked, then stopped after reboot.**
Port number may have changed. USB MIDI ports are numbered by insertion order. Zynthian's autoconnect re-wires on each boot, but if you have multiple devices, unplug others to isolate.

**Bluetooth MIDI won't pair — "AuthenticationCanceled".**
Device timed out. Pre-type the `pair XX:XX:XX:XX:XX:XX` command, activate pairing mode on the BLE device immediately, then press Enter within 3 seconds. See [MIDI Controllers](midi.md).

**Program Change messages don't switch presets.**
PC messages switch presets within the current bank. Use Bank Select (CC 0 + CC 32) first if you need a different bank, then send PC.

---

## Display

**Screen is black / no signal.**
Pi 4 has two HDMI ports. Use **HDMI0** — the port closest to the USB-C power connector. Replug (no reboot needed).

**Screen shows "error" with a character image and red text.**
JACK is not running. See "Why do I hear no sound?" above.

**UI shows a white and blue area only (no menus).**
Zynthian UI is loading. Wait 30–60 seconds after first boot. If it stays like this, check `systemctl status zynthian --no-pager` via SSH.

**Touch input not working on HDMI TV.**
HDMI TVs have no touch capability. Set wiring layout to `TOUCH_ONLY` in webconf → Hardware → Wiring, and use a USB mouse.

---

## System

**webconf is not loading at `http://zynthian.local`.**
Check `systemctl status zynthian-webconf --no-pager` via SSH. Most common cause: `libzyncore.so` missing. Fix: `cd /zynthian/zyncoder && mkdir -p build && cd build && cmake .. && make -j4`, then restart webconf.

**`zynthian.local` hostname doesn't resolve.**
On Windows, install Bonjour or use the IP address directly. Find the IP on your router's DHCP page, or read it from the Zynthian screen top bar.

**webconf password is wrong.**
Default password: `opensynth`. If changed and forgotten: SSH in as root (password `opensynth`) and reset via webconf → System → Security.

**SD card keeps corrupting.**
Poor-quality SD card. Use SanDisk Ultra or Samsung EVO. Check power: `vcgencmd get_throttled` — `0x50000` means past undervoltage. Use a 5V/3A USB-C supply.

**Software update hangs.**
Check `journalctl -u zynthian-webconf -f` — update runs via webconf. If it hangs on a specific repo, SSH in and run `update_zynthian.sh` manually.

**Zynthian loses state after power-off.**
By design — use snapshots to save state. If you want a specific state to load on every boot, save it as `last_state` (Snapshots → Save As → `last_state`). See [Snapshots](snapshots.md).

---

## Engines

**Engine won't load / shows error.**
Usually a missing dependency. Check `journalctl -u zynthian -n 50 --no-pager | grep -i error`. For LV2 plugins, run webconf → Engines → Regenerate Cache.

**ZynAddSubFX has no sound after loading a preset.**
Check MIDI channel assignment and that the engine's JACK port is connected. Restart zynautoconnect: restart zynthian service.

**FluidSynth sounds thin / no reverb.**
Reverb and chorus are disabled by default. In the engine controls (turn encoder 3 on V5 to navigate parameters), enable reverb and set room size.

**Pianoteq won't start.**
Requires a valid license. The `pianoteq` binary must be present at `/zynthian/zynthian-sw/pianoteq/`. Check `ls /zynthian/zynthian-sw/` and that the license file exists.

---

## What's Next

- [Troubleshooting](troubleshooting.md) — step-by-step diagnosis
- [Audio Setup](audio.md) — JACK configuration
- [MIDI Controllers](midi.md) — controller connection

---

*Version: 2026-05-25*
