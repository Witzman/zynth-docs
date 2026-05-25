# Zynthian — User Guide

Zynthian is an open-source synthesizer platform running on Raspberry Pi. This guide covers everything a new user needs to get started and become productive.

---

## Guide Tracks

### Get Started
For users who just received hardware or flashed a fresh SD card.

| Page | What it answers |
|------|----------------|
| [Getting Started](getting-started.md) | Flash SD card, first boot, connect to the device, set up audio and display |
| [Understanding Zynthian](userguide.md) | What Zynthian is, how to think about chains, engines, and snapshots |
| [Hardware Setup](hardware.md) | Display options, wiring layouts, supported hardware variants |

### Using the UI
Screen-by-screen guides for the Zynthian interface.

| Page | What it answers |
|------|----------------|
| [UI Navigation](ui-navigation.md) | Screen map, navigation patterns, status bar, main menu |
| [Chains & Routing](chain-management.md) | Add chains, engine flow, chain manager, chain options, audio routing |
| [Control Screen](control-screen.md) | Parameter knobs, page navigation, MIDI CC learn, XY controller |
| [MIDI CC Learning](midi-cc-learn.md) | Bind controller knobs to parameters, CC routing, MIDI profiles |
| [ZS3 Subsnapshots](zs3-guide.md) | Sub-snapshots, Program Change recall, live arrangement workflow |
| [Pattern Editor](pattern-editor.md) | Step sequencer, note grid, per-note velocity/offset, CC automation |
| [Admin & System](admin-guide.md) | WiFi, Bluetooth, brightness, updates, power |

### Play & Create
For users who have a working Zynthian and want to make music.

| Page | What it answers |
|------|----------------|
| [Synth Engines](synth-engines.md) | Which engines are available, when to use each, key parameters |
| [MIDI Controllers](midi.md) | Connect USB and Bluetooth MIDI controllers, configure MIDI routing |
| [Snapshots](snapshots.md) | Save and restore complete setups, use the snapshot library |
| [Common Setups (Recipes)](recipes.md) | Layered piano+strings, Hammond organ, live looper, drums+bass+lead |

### Configure
For users who want to tune audio, MIDI, and system settings.

| Page | What it answers |
|------|----------------|
| [Audio Setup](audio.md) | Select audio device, configure JACK, tune buffer size |
| [Webconf Reference](webconf.md) | All webconf sections explained |
| [Configuration Reference](configuration-reference.md) | All `zynthian_envars.sh` variables annotated |
| [LV2 Plugins](lv2-plugins.md) | Install, manage, and use LV2 plugins |

### Troubleshoot
When something doesn't work.

| Page | What it answers |
|------|----------------|
| [FAQ](faq.md) | Quick answers — no sound, MIDI not responding, display black, webconf down |
| [Troubleshooting](troubleshooting.md) | Step-by-step diagnosis: JACK errors, SD card corruption, USB audio, display |
| [Performance & CPU](performance-monitoring.md) | CPU load, XRUNs, DPM meters, engine cost table, optimization strategies |

### Under the Hood
For curious users and developers.

| Page | What it answers |
|------|----------------|
| [Architecture](architecture.md) | Boot sequence, systemd services, three-repo structure, MIDI routing |
| [Glossary](glossary.md) | Technical terms explained |
| [Co-developing with Claude](codevelop.md) | This project's Claude Code workflow |

---

*Version: 2026-05-25.*
