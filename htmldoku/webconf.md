# Webconf Reference

The webconf tool at `http://zynthian.local` (default password: `opensynth`) is the primary configuration interface. This page describes each section.

---

## Dashboard

The main dashboard shows system status: CPU usage, memory, temperature, JACK status, and connected MIDI devices. It also provides quick links to the most common configuration sections.

The dashboard is implemented in `zynthian-webconf/lib/dashboard_handler.py`.

---

## Hardware

### Audio

Select the audio device (soundcard), sample rate, JACK buffer size, and number of periods. Changes require a reboot. See [Audio Setup](audio.md) for guidance on values.

Handler: `zynthian-webconf/lib/audio_config_handler.py`.

### Display

Set the display type (HDMI, touchscreen model, framebuffer device) and resolution. For HDMI TVs, `HDMI` mode is standard. For official Zynthian screens, select the matching model (e.g. `ZynScreen 3.5`).

Handler: `zynthian-webconf/lib/display_config_handler.py`.

### Wiring

Select the hardware wiring layout — which GPIO pins correspond to which controls (encoders, buttons, LEDs). Use `TOUCH_ONLY` for a Pi with no physical controllers (mouse/touchscreen only).

Wiring profiles are defined in `zynthian-sys/config/wiring-profiles/`.

### Hardware Options

Additional hardware settings: MIDI DIN enable, CV/gate ports, fan control, RTC module. Handler: `zynthian-webconf/lib/hwoptions_config_handler.py`.

---

## MIDI

### Ports

Lists all detected MIDI ports (USB, Bluetooth, network). Toggle each port active/inactive. Active ports are routed to the ZynMidiRouter.

### CC Routing

Map incoming MIDI CC messages to internal controls. Assign a CC number to a Zynthian parameter.

### Profiles

Save and load MIDI routing configurations as profiles.

Handler: `zynthian-webconf/lib/midi_config_handler.py`.

---

## Engines

Shows installed LV2 plugins and native engines. Use this page to regenerate the engine cache after installing new plugins.

Handler: `zynthian-webconf/lib/engines_handler.py`.

---

## Presets

Browse and manage preset banks for FluidSynth soundfonts and other engines. Upload new `.sf2` files here.

Handler: `zynthian-webconf/lib/presets_config_handler.py`.

---

## Snapshots

List, load, save, delete, and download snapshots. Uploading a `.zss` file here adds it to the snapshot library.

---

## System

### Software Update

Pulls the latest code from all three repositories and restarts services. Equivalent to running `update_zynthian.sh` via SSH.

### Reboot / Shutdown

Clean reboot or shutdown of the Pi.

### Security

Change the webconf password. Note: this is separate from the SSH root password.

Handler: `zynthian-webconf/lib/security_config_handler.py`.

### Wifi

Configure WiFi networks. All WiFi management is done here — not via system config files directly.

Handler: `zynthian-webconf/lib/wifi_config_handler.py`.

---

## Remote Access

- **SSH:** `ssh root@zynthian.local` — password `opensynth` (default)
- **SFTP:** Same credentials, same host — use any SFTP client to transfer files
- **VNC:** Available on port 5900 if enabled in webconf
- **Browser:** `http://zynthian.local` — webconf on port 80

---

## What's Next

- [Hardware Setup](hardware.md) — wiring and display options
- [Audio Setup](audio.md) — audio device configuration
- [MIDI Controllers](midi.md) — port configuration

---

*Version: 2026-05-25 — derived from `zynthian-webconf/lib/`, `zynthian-webconf/zynthian_webconf.py`.*
