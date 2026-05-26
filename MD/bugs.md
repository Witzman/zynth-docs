# Bugs

## Open

### BLE MIDI broken on kernel 6.12 (zynbluez 5.76)

**Symptom:** `bluetoothctl connect` fails with `le-connection-abort-by-local`. BlueZ debug shows `att_connect_cb() connect to ...: Function not implemented (38)`.

**Root cause:** zynbluez 5.76 was compiled in May 2024 against kernel 6.6 headers. The L2CAP ATT socket interface changed in kernel 6.10+. The Pi now runs 6.12.47+rpt-rpi-v8.

**Workaround:** Connect SMC-PAD via USB-C instead of BLE. JACK `-X raw` handles USB MIDI natively — device appears as `system:midi_capture_N` automatically.

**Fix:** Rebuild zynbluez 5.76 from source against kernel 6.12 headers, or wait for Zynthian package update.

**Affects:** SMC-PAD BLE connection (and any other BLE MIDI device).

### JACK fails when U46DJ not connected

**Symptom:** `systemctl status jack2` shows `exit-code` / `EXCEPTION` on boot if U46DJ is not plugged in or powered on. Zynthian starts but has no audio. `journalctl -u jack2` shows `Cannot open device hw:U46DJ`.

**Root cause:** jack2.service ExecStart is configured for `hw:U46DJ` (done during U46DJ audio setup). JACK cannot open a device that is absent.

**Workaround:** Power on and connect U46DJ before booting, or before running `systemctl start jack2`.

**Affects:** All tutorials that require audio output (Drone Synth, MIDI Channel Routing, Audio FX, Performance Rig).

---

### E-MU Xboard25 ALSA client name unstable across reboots

**Symptom:** After some reboots, the Xboard appears in `aconnect -l` as `USB Device 0x41e:0x3f00` instead of `E-MU Xboard25`. The device still works — only the displayed name differs.

**Root cause:** USB enumeration order or kernel driver path determines which name string is used. Not consistent.

**Workaround:** Use `lsusb | grep 041e` to confirm the device is connected, regardless of the name shown in `aconnect -l`.

**Affects:** Any tutorial step that asks the user to identify the Xboard by name in `aconnect -l`.

---

### SINCO (SMC-PAD) card number changes across reboots

**Symptom:** The card number in `aconnect -l` (e.g. `card=3`) differs between sessions. `amidi -p hw:X,0,1` requires the current card number each time.

**Root cause:** USB device enumeration order is not fixed — card numbers are assigned dynamically at boot.

**Workaround:** Always run `aconnect -l` at the start of a session to find the current card number before using `amidi`.

**Affects:** SMC-PAD Launcher Control tutorial Part 1 Step 3.

---

## Closed

