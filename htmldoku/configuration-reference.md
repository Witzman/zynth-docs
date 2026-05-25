# Configuration Reference

All Zynthian configuration lives in `/zynthian/config/zynthian_envars.sh` (auto-generated on first boot from a hardware profile, then editable via webconf). This page documents the most important variables.

The file is sourced by `config-on-boot.sh` [`zynthian-sys/sbin/config-on-boot.sh`] on every boot before JACK starts. Changes made in webconf are written here and take effect after reboot.

To override on boot without changing the profile, put overrides in `/boot/zynthian_envars.sh` — `config-on-boot.sh` sources this file first if it exists.

---

## System

| Variable | Default (V5) | Description |
|----------|-------------|-------------|
| `ZYNTHIAN_KIT_VERSION` | `V5` | Hardware kit identifier. Set by autoconfig. Do not change manually. |
| `ZYNTHIAN_OVERCLOCKING` | `Maximum` | Pi CPU overclock profile. `Maximum` raises clock speed for better DSP performance on Pi 4. |
| `ZYNTHIAN_WIFI_MODE` | `off` | `off`, `hotspot`, or `managed`. `hotspot` creates a `ZynthianAP` access point. |
| `ZYNTHIAN_LIMIT_USB_SPEED` | `0` | Set `1` to force USB 2.0 speed on all ports. Workaround for devices with USB 3.0 issues. |
| `ZYNTHIAN_DISABLE_OTG` | `0` | Set `1` to disable USB OTG port (Pi 4 USB-C). |
| `BOOTLOG` | `0` | Set `1` to keep full boot log visible on screen. |

---

## Audio

| Variable | Default (V5) | Description |
|----------|-------------|-------------|
| `SOUNDCARD_NAME` | `V5 ADAC` | Human-readable audio device name shown in webconf. |
| `SOUNDCARD_CONFIG` | `dtoverlay=hifiberry-dacplusadcpro\n...` | Device tree overlay string added to `/boot/firmware/config.txt`. |
| `JACKD_OPTIONS` | `-P 70 -s -S -d alsa -d hw:sndrpihifiberry -r 48000 -p 256 -n 2 -o 2 -i 2` | Full jackd command line. Key flags: |
| | | `-d hw:NAME` — ALSA card name to use |
| | | `-r RATE` — sample rate (44100 or 48000) |
| | | `-p SIZE` — buffer frames (128/256/512) |
| | | `-n PERIODS` — periods per cycle (2 or 3) |
| | | `-o OUTPUTS` — number of output channels |
| | | `-i INPUTS` — number of input channels |
| `ZYNTHIAN_DISABLE_RBPI_AUDIO` | `0` | Set `1` to disable the built-in headphone jack in ALSA (prevents it from appearing as `hw:0`). |
| `ZYNTHIAN_RBPI_HEADPHONES` | `0` | Set `1` to route audio through the built-in headphone jack. Not recommended for performance use. |

---

## Display

| Variable | Description |
|----------|-------------|
| `DISPLAY_NAME` | Display model name. Matches an entry in the webconf display list. |
| `DISPLAY_CONFIG` | Device tree overlay config lines for the display. |
| `DISPLAY_WIDTH` / `DISPLAY_HEIGHT` | Resolution in pixels. |
| `DISPLAY_ROTATION` | `Normal`, `Inverted`, `Left`, `Right`. |
| `FRAMEBUFFER` | Linux framebuffer device (e.g. `/dev/fb0`, `/dev/fb1`). |
| `DISPLAY_KERNEL_OPTIONS` | Added to kernel command line (e.g. `video=DSI-1:800x480@60`). |

---

## Wiring

| Variable | Description |
|----------|-------------|
| `ZYNTHIAN_WIRING_LAYOUT` | Hardware wiring profile. Common values: `V5`, `Z2`, `TOUCH_ONLY`, `PROTOTYPE-4`. |
| `ZYNTHIAN_WIRING_ENCODER_A` / `_B` | GPIO pin numbers for encoder signals (only needed for custom wiring). |
| `ZYNTHIAN_WIRING_SWITCHES` | GPIO pin list for button inputs. |
| `ZYNTHIAN_WIRING_MCP23017_I2C_ADDRESS` | I2C address of MCP23017 GPIO expander (V5: `0x20,0x21`). |
| `ZYNTHIAN_WIRING_LAYOUT_CUSTOM_PROFILE` | Which wiring-profile directory to load for custom switch mappings. |
| `ZYNTHIAN_WIRING_CUSTOM_SWITCH_NN__UI_SHORT` | What UI action a short-press of switch NN triggers. |
| `ZYNTHIAN_WIRING_CUSTOM_SWITCH_NN__UI_LONG` | What UI action a long-press triggers (≥ `ZYNTHIAN_UI_SWITCH_LONG_MS`). |
| `ZYNTHIAN_WIRING_CUSTOM_SWITCH_NN__UI_BOLD` | What UI action a bold-press triggers (≥ `ZYNTHIAN_UI_SWITCH_BOLD_MS`). |
| `ZYNTHIAN_WIRING_CUSTOM_SWITCH_NN__MIDI_CHAN` | MIDI channel for switch MIDI output (`Active` = current chain channel). |
| `ZYNTHIAN_WIRING_CUSTOM_SWITCH_NN__MIDI_NUM` | MIDI CC number for switch output. |
| `ZYNTHIAN_WIRING_CUSTOM_SWITCH_NN__MIDI_VAL` | MIDI CC value for switch output. |

**Common UI_SHORT/UI_LONG actions:**

| Action | Effect |
|--------|--------|
| `MENU` | Open main menu |
| `BACK` | Go back / close |
| `SCREEN_ADMIN` | Open Admin screen |
| `SCREEN_AUDIO_MIXER` | Open ALSA mixer |
| `SCREEN_SNAPSHOT` | Open Snapshot screen |
| `SCREEN_LAUNCHER` | Open screen launcher |
| `TOGGLE_RECORD` | Start/stop MIDI recording |
| `TOGGLE_PLAY` | Play/stop MIDI playback |
| `ALL_NOTES_OFF` | Send all-notes-off on all channels |
| `ALL_SOUNDS_OFF` | Kill all audio immediately |
| `POWER_OFF` | Shutdown the Pi |
| `PROGRAM_CHANGE N` | Send Program Change message N on active chain |
| `PRESET_FAV` | Toggle preset as favourite |
| `CHAIN_CONTROL` | Open chain control screen |
| `BANK_PRESET` | Open bank/preset screen |

---

## UI

| Variable | Default | Description |
|----------|---------|-------------|
| `ZYNTHIAN_UI_RESTORE_LAST_STATE` | `1` | Auto-load `last_state.zss` on startup. Set `0` to start empty. |
| `ZYNTHIAN_UI_SNAPSHOT_MIXER_SETTINGS` | `1` | Include mixer levels in snapshots. |
| `ZYNTHIAN_UI_SWITCH_BOLD_MS` | `300` | Milliseconds for a button press to register as "bold". |
| `ZYNTHIAN_UI_SWITCH_LONG_MS` | `2000` | Milliseconds for a button press to register as "long". |
| `ZYNTHIAN_UI_ENABLE_CURSOR` | `0` | Show mouse cursor. Set `1` when using a mouse on HDMI. |
| `ZYNTHIAN_UI_POWER_SAVE_MINUTES` | `10` | Screen blank after N minutes idle. `0` = disable. |
| `ZYNTHIAN_UI_MULTICHANNEL_RECORDER` | `1` | Record each chain to a separate audio file. |
| `ZYNTHIAN_UI_VISIBLE_MIXER_STRIPS` | `0` | How many mixer strips to show (`0` = auto). |
| `ZYNTHIAN_UI_TOUCH_NAVIGATION` | `` | Set to enable touch navigation mode. |
| `ZYNTHIAN_VNCSERVER_ENABLED` | `0` | Enable VNC server on port 5900. Set `1` and reboot. |
| `ZYNTHIAN_MIDI_PLAY_LOOP` | `1` | Loop MIDI playback. |
| `ZYNTHIAN_UI_COLOR_BG` | `#000000` | UI background color (hex). |
| `ZYNTHIAN_UI_COLOR_TX` | `#ffffff` | UI text color. |
| `ZYNTHIAN_UI_COLOR_ON` | `#ff0000` | UI "active" accent color. |
| `ZYNTHIAN_UI_FONT_FAMILY` | `Audiowide` | UI font family. |
| `ZYNTHIAN_UI_FONT_SIZE` | `16` | UI base font size in pixels. |

---

## MIDI

| Variable | Default | Description |
|----------|---------|-------------|
| `ZYNTHIAN_SCRIPT_MIDI_PROFILE` | `/zynthian/config/midi-profiles/default.sh` | Path to MIDI routing profile script. |
| `ZYNTHIAN_USB_MIDI_BY_PORT` | `0` | Set `1` to identify USB MIDI devices by port number (stable) rather than device name. |

---

## Directory Paths

All paths are set in the envars file and should not be changed without understanding the full dependency chain.

| Variable | Value | Purpose |
|----------|-------|---------|
| `ZYNTHIAN_DIR` | `/zynthian` | Root of all Zynthian repos and data |
| `ZYNTHIAN_UI_DIR` | `/zynthian/zynthian-ui` | UI source |
| `ZYNTHIAN_SYS_DIR` | `/zynthian/zynthian-sys` | System scripts |
| `ZYNTHIAN_DATA_DIR` | `/zynthian/zynthian-data` | Read-only factory data (soundfonts, snapshots) |
| `ZYNTHIAN_MY_DATA_DIR` | `/zynthian/zynthian-my-data` | User data (snapshots, recordings, uploads) |
| `LV2_PATH` | Multiple paths | Where JACK/jalv searches for LV2 plugins |

---

## Applying Changes

Most variables can be changed in webconf — it writes the envars file and prompts for reboot.

To change directly via SSH:

```bash
# Edit the file
nano /zynthian/config/zynthian_envars.sh

# Reboot to apply
reboot
```

JACK-related changes (sample rate, buffer size, card name) always require a reboot. UI color/font changes take effect on zynthian service restart without full reboot:

```bash
systemctl restart zynthian
```

---

## What's Next

- [Hardware Setup](hardware.md) — hardware-specific env vars
- [Audio Setup](audio.md) — JACKD_OPTIONS tuning
- [Webconf Reference](webconf.md) — GUI for most of these settings

---

*Version: 2026-05-25 — derived from `zynthian-sys/config/zynthian_envars_V5.sh`, `zynthian-sys/sbin/config-on-boot.sh`.*
