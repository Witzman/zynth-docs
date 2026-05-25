# MIDI Controllers

Zynthian accepts MIDI from USB devices, Bluetooth LE MIDI, DIN-5 ports (on Zynthian kit hardware), and networked MIDI. This page explains how to connect each type.

---

## USB MIDI Controllers

USB MIDI is plug-and-play. Connect a USB keyboard, pad controller, or interface to any USB port on the Pi. Zynthian detects it automatically on boot and on hot-plug.

To verify detection, SSH in and run:

```bash
aconnect -l
```

The controller appears as a MIDI client (e.g. `client 32: 'SINCO' [type=kernel,card=4]`). Zynthian's ZynMidiRouter connects all detected MIDI inputs to the active chains automatically [`zynthian-ui/zyngine/zynthian_engine_midi_control.py`].

To configure which ports route where, open `http://zynthian.local` → **MIDI** → **Ports**. Enable each port you want active.

---

## MIDI Channel Routing

By default, MIDI channel 1 drives the first chain, channel 2 the second, and so on. You can change channel assignments per chain in the UI:

1. Tap the chain in the layer view.
2. Open chain options → **MIDI Channel**.
3. Select the incoming channel (or **Omni** to respond to all channels).

---

## Bluetooth MIDI

Bluetooth MIDI uses the BLE MIDI profile. Zynthian supports it via the system bluetooth service.

**Pair a BLE MIDI device:**

```bash
systemctl start bluetooth
bluetoothctl power on
bluetoothctl scan on
# Put your device in pairing mode — it appears as:
# [NEW] Device XX:XX:XX:XX:XX:XX DeviceName
bluetoothctl pair XX:XX:XX:XX:XX:XX
bluetoothctl trust XX:XX:XX:XX:XX:XX
bluetoothctl connect XX:XX:XX:XX:49:23:90
```

After pairing, check `aconnect -l` for the device's MIDI port. Some BLE MIDI devices disconnect immediately after pairing and reconnect when you play — this is normal.

> **Note:** Bluetooth MIDI pairing can fail with `AuthenticationCanceled` if the device times out. Put the device in pairing mode immediately before running `pair`. The `trust` command ensures automatic reconnection on future boots.

---

## DIN-5 MIDI (Hardware Kit Only)

Zynthian kit hardware includes physical DIN-5 MIDI IN/THRU/OUT ports connected to the Pi's UART. These work out of the box on official kits. On a custom build, enable the MIDI UART in webconf: **Hardware** → **MIDI Options** → enable MIDI DIN.

---

## Network MIDI

Zynthian supports QMidiNet (for network MIDI across devices on the same LAN). Enable it via webconf: **MIDI** → **Network** → enable QMidiNet.

---

## MIDI Filtering and CC Mapping

Open `http://zynthian.local` → **MIDI** → **CC** to map incoming MIDI CC messages to Zynthian controls. Useful for assigning knobs on a controller to synth parameters.

Program Change (PC) messages switch presets. Bank Select (CC 0/32) switches banks.

---

## Troubleshooting MIDI

**Controller not detected:**
```bash
dmesg | grep -i "usb\|midi" | tail -10
aconnect -l
```
If the device shows in `dmesg` but not `aconnect -l`, it may need a driver or udev quirk.

**Notes play but wrong engine responds:** Check the MIDI channel setting on the chain versus what channel the controller sends on. Use a MIDI monitor to verify:
```bash
amidi -p hw:X,0,0 -d    # replace X with card number
```

**Bluetooth device disconnects repeatedly:** The BLE MIDI reconnection is handled by `bluetoothctl` autoconnect. Run `bluetoothctl trust XX:XX:XX:XX:XX:XX` if not already done.

---

## What's Next

- [Synth Engines](synth-engines.md) — choose an engine to play
- [Snapshots](snapshots.md) — save the routing setup
- [Audio Setup](audio.md) — configure audio output

---

*Version: 2026-05-25 — derived from `zynthian-ui/zyngui/zynthian_gui_bluetooth.py`, `zynthian-webconf/lib/midi_config_handler.py`.*
