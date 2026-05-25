# Admin & System Screens

The Admin screen provides system configuration, MIDI settings, audio options, network management, and hardware controls. Most items here affect global behavior rather than per-chain settings.

---

## Accessing Admin

| Method | How |
|--------|-----|
| Main Menu | Main Menu (SW1 short) → **Admin** |
| V5 hardware | SW1 long hold → SCREEN_ADMIN |
| From chain_control | SW2 bold press → Main Menu → Admin |

---

## Admin Screen Structure

The admin list is organized in sections. Section headers (shown in a highlight color) group related options.

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

| Option | Range | Effect | Env var |
|--------|-------|--------|---------|
| Visible Chains (N) | 4–16 | How many chain strips appear in the mixer | `ZYNTHIAN_UI_VISIBLE_MIXER_STRIPS` |
| Visible Launchers (N) | 4–16 | How many launcher rows appear in the clip view | `ZYNTHIAN_UI_VISIBLE_LAUNCHERS` |

Both take effect immediately — the mixer layout redraws without restart.

---

## MIDI Section

| Option | State | Effect | Env var |
|--------|-------|--------|---------|
| **MIDI Input Devices** | — | Open midi_config in admin mode — all input ports | — |
| **MIDI Output Devices** | — | Open midi_config output view — all output ports | — |
| **Active MIDI channel** | ☑/☐ | ☑ = MIDI input to all chains on that channel; ☐ = active chain only | `ZYNTHIAN_MIDI_SINGLE_ACTIVE_CHANNEL` |
| **Program Change for ZS3** | ☑/☐ | ☑ = PC messages recall ZS3 states; ☐ = PC loads engine presets | `ZYNTHIAN_MIDI_PROG_CHANGE_ZS3` |
| **MIDI Bank Change** | ☑/☐ | ☑ = Bank Select (CC 0/32) processed; only visible when ZS3 mode off | `ZYNTHIAN_MIDI_BANK_CHANGE` |
| **MIDI-USB mapped by port** | ☑/☐ | ☑ = USB MIDI device ID includes physical port number | `ZYNTHIAN_MIDI_USB_BY_PORT` |
| **Global Transpose (±N)** | −24 to +24 | Transpose all MIDI notes globally (semitones), additive to per-chain transpose | via `lib_zyncore` |
| **Channel Pressure → CC (N)** | 0–119 or NONE | Map mono aftertouch to the selected CC number | `ZYNTHIAN_MIDI_CHANPRESS_CC` |

### Active MIDI Channel Detail

When **Active MIDI channel** is enabled (☑):
- CC, pitch bend, and PC messages from any MIDI input are routed to **all chains** assigned to the same MIDI channel.
- Useful for multi-timbral setups where one keyboard drives all chains simultaneously.

When disabled (☐):
- MIDI messages go to the **currently selected (active)** chain only, regardless of which chains share the channel.

### USB Mapped by Port Detail

When **MIDI-USB mapped by port** is enabled (☑):
- Two identical USB MIDI devices plugged into different USB ports are treated as different devices.
- Prevents device ID collisions when using multiple identical controllers (e.g. two Akai MPK Mini).

When disabled (☐):
- Devices are identified by name only — two identical devices share the same MIDI port.

---

## AUDIO Section

| Option | State | Effect | Env var |
|--------|-------|--------|---------|
| **Audio Levels** | — | Open ALSA mixer (alsa_mixer screen) | — |
| **ZynVoice (text to speech)** | — | Open TTS accessibility options (tts screen) | — |
| **RBPi Headphones** | ☑/☐ | Enable Pi onboard 3.5mm audio (low fidelity); starts `headphones.service` | `ZYNTHIAN_RBPI_HEADPHONES` |
| **Hotplug USB Audio** | ☑/☐ + sub-options | Enable auto-detection of USB audio devices; configure which devices are enabled | `ZYNTHIAN_HOTPLUG_AUDIO` |
| **Audio Levels on Snapshots** | ☑/☐ | ☑ = soundcard ALSA mixer values saved with snapshots | `ZYNTHIAN_UI_SNAPSHOT_MIXER_SETTINGS` |
| **Mixer Peak Meters** | ☑/☐ | ☑ = DPM meters visible in mixer strips; ☐ = hidden (saves ~1–2% CPU) | `ZYNTHIAN_UI_ENABLE_DPM` |

### Hotplug USB Audio Sub-Options

When hotplug audio is enabled, a sub-menu appears:

| Option | Effect |
|--------|--------|
| ☑/☐ Hotplug Audio | Master enable/disable for USB audio hotplug |
| **Input Devices** section | List of detected USB audio inputs |
| ☑/☐ `device` in | Enable/disable this USB input device |
| **Output Devices** section | List of detected USB audio outputs |
| ☑/☐ `device` out | Enable/disable this USB output device |

If no outputs are available and ZynVoice is configured, an option appears to navigate to the TTS settings.

### RBPi Headphones

The Pi's onboard 3.5mm jack is a low-fidelity output (10-bit DAC). Use it only for monitoring when no proper audio interface is connected, or for quiet reference listening. Enabling it starts a separate JACK client that duplicates the main output to the Pi's audio subsystem.

---

## NETWORK Section

| Option | Detail |
|--------|--------|
| **Network Info** | Shows current IP addresses, hostname, and network interface status |
| **Wi-Fi Config (status)** | Opens wifi screen — current connection status in parentheses |
| **VNC Server** ☑/☐ | ☑ = VNC desktop sharing active; starts `vncserver0` service |

### Wi-Fi Screen

The wifi screen scans continuously (1-second interval background thread) and shows all detectable networks.

**Top toggle:**

| State | Display | Action |
|-------|---------|--------|
| Wi-Fi disabled | `☐ Wi-Fi is disabled` | Tap to enable |
| Wi-Fi enabled | `☑ Wi-Fi is enabled` | Tap to disable |

**Below toggle — network list:**

| Entry | Meaning | Tap action |
|-------|---------|-----------|
| "Scanning Wi-Fi Networks..." | No results yet | Wait |
| Network name (bold) | Known and connected | Tap to disconnect |
| Network name | Known, not connected | Tap to reconnect |
| Network name (new) | Not configured | Tap → keyboard for password |

Password entry uses the on-screen keyboard. Failed connections prompt for re-entry with the password cleared.

If connecting fails with the saved password, the screen prompts for a new password, deletes the old connection, and creates a new one.

Source: [`zyngui/zynthian_gui_wifi.py`](../zynthian-ui/zyngui/zynthian_gui_wifi.py)

### VNC Server

When VNC is enabled, you can connect from any VNC client on the same network:
- **Host:** `zynthian.local` or the IP shown in the status bar
- **Port:** 5900 (standard VNC)
- **Displays:** Zynthian GUI + any engine native GUI (e.g. ZynAddSubFX synth editor)

VNC uses extra CPU (~5–8%). Disable during performance-critical sessions.

---

## SETTINGS Section

| Option | Detail | Env var |
|--------|--------|---------|
| **Preset Preload** ☑/☐ | ☑ = presets pre-load while browsing list (faster audition, more CPU); ☐ = load only on select | `ZYNTHIAN_UI_PRESET_PRELOAD` |
| **Touch Navigation** | Select touch interaction mode | `ZYNTHIAN_UI_TOUCH_NAVIGATION` |
| **Brightness** | Display and LED brightness sliders | — |
| **CV Settings** | Zynaptik CV/gate configuration | — |
| **Calibrate Touchscreen** | Touch calibration wizard | — |
| **Clean Screen** | 10s locked countdown for screen cleaning | — |
| **Bluetooth** | Open bluetooth screen | — |

### Touch Navigation Options

| Option | Behavior |
|--------|---------|
| **None** | Standard navigation — encoders and touch on full-screen UI |
| **V5 keypad at left** | On-screen V5-style encoder simulation on left edge |
| **V5 keypad at right** | On-screen V5-style encoder simulation on right edge |

Changing Touch Navigation requires a UI restart (prompt appears to confirm).

### Brightness Screen

Available when `brightness_config` screen has controls (display brightness hardware present). Shows sliders:

| Slider | Controls |
|--------|---------|
| Display brightness | Backlight intensity (PWM) |
| LED brightness | Encoder ring LEDs (V5 hardware only) |

Changes take effect immediately. No restart required.

### CV Settings

Only shown when a Zynaptik expansion board is detected. Configures:

| Setting | Purpose |
|---------|---------|
| CV input assignment | Which chain receives CV pitch input |
| Gate input assignment | Which chain's note on/off is triggered by gate |
| CV output assignment | Which chain's MIDI pitch feeds CV output |
| CV output scaling | V/Oct calibration |

### Touchscreen Calibration

Opens a calibration overlay: tap each crosshair in sequence (usually 5 points). The screen automatically closes after 15 seconds of inactivity. Use when touch registration is offset — typically after a display replacement or first boot on a new display.

---

## Bluetooth Screen

### List Structure

**Enable toggle:**

| State | Display | Action |
|-------|---------|--------|
| BT disabled | `☐ Enable Bluetooth` | Tap to start bluetooth service and begin scanning |
| BT enabled | `☑ Enable Bluetooth` | Tap to stop bluetooth service |

**Controllers section (when enabled):**

One entry per detected Bluetooth controller (internal Pi BT chip or USB BT dongle):

| Entry format | Meaning |
|-------------|---------|
| `  ☑ Controller Name` | This controller is powered on and active |
| `  ☐ Controller Name` | This controller is present but powered off |

Tap to toggle power on/off. Only one controller can be active at a time. Bold-press to rename the controller.

**Devices section:**

All detected BLE devices nearby:

| Entry format | Meaning |
|-------------|---------|
| `☑ ⚡ DeviceName` | Trusted, connected (BLE MIDI device, connected icon visible) |
| `☑ DeviceName` | Trusted, not currently connected |
| `☐ DeviceName` | Not trusted (seen during scan but not paired) |

| Action | Result |
|--------|--------|
| Tap ☐ device | Trust + pair + connect → device becomes available as MIDI port |
| Tap ☑ device | Untrust + disconnect + remove from paired list |
| Bold-press ☑ device | Confirm removal dialog |

Once trusted, BLE MIDI devices appear as ALSA MIDI ports and are automatically routed through ZynMidiRouter on the next connection.

> **Important:** The Pi's onboard Bluetooth adapter has limited range and occasional connection issues. A USB Bluetooth 4.0+ dongle (any standard dongle) gives much better reliability. Plug in before booting — it appears as a second controller in the list and can be selected instead of the built-in adapter.

Source: [`zyngui/zynthian_gui_bluetooth.py`](../zynthian-ui/zyngui/zynthian_gui_bluetooth.py)

---

## TEST Section

| Option | Detail |
|--------|--------|
| **Test Audio** | Plays a short test audio file through the main output — confirms audio routing is working |
| **Test MIDI** | Plays a MIDI file through all loaded chains — confirms MIDI routing and engine response |
| **Test control HW** | Hardware encoder and button test (only available in debug builds) |

Press **Back** during Test Audio or Test MIDI to stop playback. Test MIDI will trigger note-on messages on all channels — if engines are loaded, they will play.

---

## SYSTEM Section

| Option | When visible | Detail |
|--------|-------------|--------|
| **Capture Workflow** ☑/☐ | Always | Record screen + audio + button actions to file for bug reports |
| **Update Software** | Only when update available (↻ in status bar) | Pull latest Zynthian code and packages from internet |
| **Power** | Always | Shutdown / reboot submenu |

### Software Update Process

The update option only appears when `state_manager.update_available` is True, indicated by the ↻ icon in the status bar. The update:

1. Pulls latest commits from Zynthian GitHub repos (`git pull`).
2. Installs any new system packages (`apt`).
3. Rebuilds C components if needed (`cmake && make`).
4. Displays a scrolling log as commands run.
5. Reboots automatically on completion.

**Duration:** 2–15 minutes depending on internet speed and size of changes.

**Do not power off during update.** Partial package installation can corrupt the system. If power fails mid-update, the next boot attempts to re-run the update.

### Power Options

| Option | Effect |
|--------|--------|
| **Shutdown** | Stops all services, unmounts filesystems, halts CPU. Power LED goes dark within ~10 seconds. Safe to unplug after LED off. |
| **Reboot** | Full software reboot. Equivalent to `systemctl reboot`. |

Always use software shutdown before removing power. Cutting power during a filesystem write can corrupt the SD card. If the card gets corrupted, see [Troubleshooting](troubleshooting.html) → SD card recovery.

---

## Env Vars Written by Admin

For reference — changes made in the Admin screen write to `zynthian_envars.sh` via `zynconf.save_config()`:

| Admin option | Env var name |
|-------------|-------------|
| Visible Chains | `ZYNTHIAN_UI_VISIBLE_MIXER_STRIPS` |
| Visible Launchers | `ZYNTHIAN_UI_VISIBLE_LAUNCHERS` |
| Active MIDI channel | `ZYNTHIAN_MIDI_SINGLE_ACTIVE_CHANNEL` |
| Program Change for ZS3 | `ZYNTHIAN_MIDI_PROG_CHANGE_ZS3` |
| MIDI Bank Change | `ZYNTHIAN_MIDI_BANK_CHANGE` |
| MIDI-USB mapped by port | `ZYNTHIAN_MIDI_USB_BY_PORT` |
| Channel Pressure → CC | `ZYNTHIAN_MIDI_CHANPRESS_CC` |
| RBPi Headphones | `ZYNTHIAN_RBPI_HEADPHONES` |
| Audio Levels on Snapshots | `ZYNTHIAN_UI_SNAPSHOT_MIXER_SETTINGS` |
| Preset Preload | `ZYNTHIAN_UI_PRESET_PRELOAD` |
| Touch Navigation | `ZYNTHIAN_UI_TOUCH_NAVIGATION` |

---

## What's Next

- [MIDI Controllers](midi.html) — connect USB and BLE MIDI devices
- [Audio Setup](audio.html) — configure the audio interface and JACK
- [Troubleshooting](troubleshooting.html) — diagnose common problems
- [Webconf Reference](webconf.html) — browser-based configuration (more options than Admin screen)
- [Configuration Reference](configuration-reference.html) — all env vars annotated

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_admin.py`, `zyngui/zynthian_gui_wifi.py`, `zyngui/zynthian_gui_bluetooth.py`, `zyngui/zynthian_gui_brightness_config.py`.*
