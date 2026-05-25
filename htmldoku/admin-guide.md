# Admin & System Screens

The Admin screen provides system configuration, MIDI settings, audio options, network management, and hardware controls. Most items here affect global behavior rather than per-chain settings.

---

## Accessing Admin

| Method | How |
|--------|-----|
| Main Menu | Main Menu → **Admin** |
| V5 hardware | SW1 hold (Long press) → SCREEN_ADMIN |
| From chain_control | SW2 bold press → Main Menu → Admin |

---

## Admin Screen Structure

The admin list is organized in sections. Each section header (shown in a different color) groups related options.

```
> MIXER
> MIDI
> AUDIO
> NETWORK
> SETTINGS
> TEST
> SYSTEM
```

Source: [`zyngui/zynthian_gui_admin.py:104`](../zynthian-ui/zyngui/zynthian_gui_admin.py)

---

## MIXER Section

| Option | Effect |
|--------|--------|
| Visible Chains (N) | How many chain strips appear in the mixer view (1–16+) |
| Visible Launchers (N) | How many clip launcher rows appear in the launcher tab |

---

## MIDI Section

| Option | Effect |
|--------|--------|
| MIDI Input Devices | Open MIDI input device configuration (port enable/disable) |
| MIDI Output Devices | Open MIDI output device configuration |
| Active MIDI channel ☑/☐ | ☑ = messages sent to all chains on same channel; ☐ = active chain only |
| Program Change for ZS3 ☑/☐ | ☑ = PC messages recall ZS3 states; ☐ = PC messages load presets |
| MIDI Bank Change ☑/☐ | Whether Bank Select messages are processed (only when ZS3 mode off) |
| MIDI-USB mapped by port ☑/☐ | ☑ = USB devices indexed by name + port number; ☐ = by name only |
| Global Transpose (±N) | Transpose all MIDI notes globally (semitones, additive to per-chain transpose) |
| Channel Pressure → CC (N) | Map monophonic aftertouch to a CC number |

---

## AUDIO Section

| Option | Effect |
|--------|--------|
| Audio Levels | Open the audio levels / DPM view full-screen |
| ZynVoice (text to speech) | Accessibility TTS options |
| RBPi Headphones ☑/☐ | Enable Pi onboard 3.5mm headphone output (low fidelity) |
| Hotplug USB Audio | Configure USB audio hotplug detection behavior |
| Audio Levels on Snapshots ☑/☐ | Whether soundcard mixer levels are saved with snapshots |
| Mixer Peak Meters ☑/☐ | Enable/disable DPM level meters in mixer (small CPU saving when off) |

---

## NETWORK Section

| Option | Effect |
|--------|--------|
| Network Info | Show IP addresses, hostname, and network status |
| Wi-Fi Config (status) | Open WiFi network browser and connector |
| VNC Server ☑/☐ | Enable/disable VNC access to the Zynthian display |

### Wi-Fi Screen

Access: Admin → **Wi-Fi Config**.

The wifi screen scans continuously (background thread, 1s interval) and shows:

- A master **Wi-Fi enabled/disabled** toggle at the top
- All detected networks below, with connection status

| Action | Result |
|--------|--------|
| Tap the enable toggle | Turn Wi-Fi radio on or off |
| Tap an unconfigured network | Keyboard screen to enter password |
| Tap a known connected network | Disconnects it |
| Tap a known disconnected network | Connects (uses saved credentials) |

Connection is managed via `nmcli`. The screen shows live Wi-Fi status ("Scanning...", "Connected", network name).

Source: [`zyngui/zynthian_gui_wifi.py`](../zynthian-ui/zyngui/zynthian_gui_wifi.py)

---

## SETTINGS Section

| Option | Effect |
|--------|--------|
| Preset Preload ☑/☐ | Pre-load presets while browsing the list (faster audition, more CPU) |
| Touch Navigation | Configure touch-widget mode for touch-only devices |
| Brightness | Display and LED brightness sliders (when `brightness_config` is available) |
| CV Settings | Zynaptik CV/gate configuration (when Zynaptik hardware is present) |
| Calibrate Touchscreen | Run the touch calibration wizard (tap 5 crosshairs) |
| Clean Screen | 10-second countdown with input locked — clean the display safely |
| Bluetooth | Open Bluetooth device management |

### Bluetooth Screen

Access: Admin → **Bluetooth**.

The bluetooth screen uses `bluetoothctl` in the background to scan and manage BLE devices.

**Structure:**
- A master **Enable Bluetooth** toggle at the top
- Controller list (usually one — the Pi's internal adapter or a USB adapter)
- Device list below

| Action | Result |
|--------|--------|
| Tap enable toggle | Start or stop the Bluetooth service |
| Tap a controller | Toggle it on/off; only one controller can be active at a time |
| Bold-press a controller | Rename it |
| Tap a device (☐) | Pair and trust the device |
| Tap a device (☑) | Untrust and disconnect |
| Bold-press a device | Confirm removal from the paired list |

BLE MIDI devices are detected by their Vendor-specific UUID (`03b80e5a-...`). Once trusted, they appear as JACK ALSA MIDI ports and are auto-connected to the ZynMidiRouter.

> The Pi's onboard Bluetooth adapter has poor range. A USB Bluetooth adapter (any USB BT 4.0+ dongle) gives much better reliability for BLE MIDI.

Source: [`zyngui/zynthian_gui_bluetooth.py`](../zynthian-ui/zyngui/zynthian_gui_bluetooth.py)

### Brightness Screen

Access: Admin → **Brightness** (only shown when brightness controls exist for this hardware).

Two or more sliders:
- **Display brightness** — backlight level
- **LED brightness** — encoder ring LEDs (V5 only)

Adjust with encoder 4 or touch. Takes effect immediately.

### CV Settings

Access: Admin → **CV Settings** (only shown when Zynaptik board is present).

Configures the CV/gate input and output assignments for the Zynaptik expansion:
- Which chains receive CV pitch
- Which gate input triggers note on/off
- Output scaling

---

## TEST Section

| Option | Effect |
|--------|--------|
| Test Audio | Plays a test audio file through the audio output |
| Test MIDI | Plays a MIDI file through the loaded chains |
| Test control HW | Hardware test for encoders and switches (debug builds only) |

Test Audio and Test MIDI cancel with Back. Test MIDI sends MIDI through any currently loaded chains — useful for checking that sound works end-to-end.

---

## SYSTEM Section

| Option | Effect |
|--------|--------|
| Capture Workflow ☑/☐ | Records screen, audio, and button actions to a file for debugging |
| Update Software | Downloads and installs Zynthian software updates (only shown when update is available) |
| Power | Shutdown / reboot submenu |

### Software Update

The **Update Software** option only appears when `state_manager.update_available` is True — signaled by the ↻ icon in the status bar.

The update process:
1. Downloads updated packages via `apt` and pulls updated Zynthian repos via `git`.
2. Shows a scrolling log of commands as they run.
3. Takes 2–10 minutes depending on connection speed and update size.
4. **Do not power off during update.** The system will reboot automatically when complete.

---

## Power

Access: Main Menu → **Power**, or Admin → **Power**.

Options: **Shutdown** and **Reboot**. Both require confirmation.

Always use the software power-off before removing power — the SD card can corrupt if power is cut during a write. After shutdown, the Pi's power LED goes dark (within ~10 seconds) and it is safe to unplug.

---

## What's Next

- [MIDI Controllers](midi.html) — connect USB and BLE MIDI devices
- [Audio Setup](audio.html) — configure the audio interface
- [Troubleshooting](troubleshooting.html) — diagnose common problems
- [Webconf Reference](webconf.html) — browser-based configuration (more options)

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_admin.py`, `zyngui/zynthian_gui_wifi.py`, `zyngui/zynthian_gui_bluetooth.py`.*
