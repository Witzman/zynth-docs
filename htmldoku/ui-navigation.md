# UI Navigation & Screen Map

This page explains how the Zynthian touch display is organized, how to reach any screen from any other, and the navigation patterns that apply everywhere.

---

## Root Screen: Mixer and Launcher

When Zynthian starts, the **Mixer** screen is the home view. It shows one vertical strip per chain — left-to-right across the display — with the master chain always at the far right.

Each strip shows:
- Chain name and engine identifier at the top
- A vertical fader for volume
- DPM level meters (when enabled)
- Mute / solo / active indicators
- MIDI channel badge

Below the chain strips, the same screen can show two tab views:

| Tab | Shows | Switch |
|-----|-------|--------|
| **Mixer** | Volume faders, DPM meters, solo/mute buttons per chain | Tap mixer label |
| **Launcher** | Clip / sequence pads — rows = phrases, columns = chains | Tap launcher label |

The mixer is the root. Pressing Back from any screen eventually returns here.

---

## Status Bar

Runs across the top of every screen:

```
┌──────────────┬──────┬──────────┬──────┬──────┬──────┐
│ 192.168.1.5  │ 35%  │ ♩ 120.0  │ ↑↓   │ 🔊   │  ↻   │
└──────────────┴──────┴──────────┴──────┴──────┴──────┘
   IP address   CPU%    Tempo      MIDI  Audio  Update
```

| Field | Detail |
|-------|--------|
| IP address | Current IP on `wlan0` or `eth0`; tap to see full network info |
| CPU% | All-core average load; above ~80% risks XRUNs |
| Tempo | Current BPM; tap opens the tempo screen |
| MIDI activity | Flashes green on incoming MIDI |
| Audio activity | Flashes on audio output signal |
| ↻ | Update available — shown only when `state_manager.update_available` is True |

---

## Main Menu

A 3×3 grid of quick-access buttons. Access via:

| Method | Trigger |
|--------|---------|
| V5 hardware | SW1 short press |
| Touchscreen | Long-press Back from most screens |
| chain_control | SW2 bold press |

| Grid position | Button | Destination |
|---------------|--------|-------------|
| Row 1, Col 1 | Chain Manager | `chain_manager` |
| Row 1, Col 2 | Snapshots | `snapshot` |
| Row 1, Col 3 | ZS3 | `zs3` |
| Row 2, Col 1 | Tempo | tempo screen |
| Row 2, Col 2 | Audio Player | `audio_player` |
| Row 2, Col 3 | MIDI Player | `midi_player` / `midi_recorder` |
| Row 3, Col 1 | Admin | `admin` |
| Row 3, Col 2 | Soundcard Levels | `alsa_mixer` |
| Row 3, Col 3 | Power | power/shutdown dialog |

Source: [`zyngui/zynthian_gui_main_menu.py:38`](../zynthian-ui/zyngui/zynthian_gui_main_menu.py)

---

## Complete Screen Reference

All screens in the Zynthian UI, grouped by category:

### Root and Navigation

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `mixer` | Main home screen — chain strips + DPM | Boot default; Back from anywhere |
| `launcher` | Clip/sequence pad view | Tab switch in mixer |
| `main_menu` | 3×3 quick-access grid | SW1 short or long-press Back |
| `chain_manager` | Visual graph of all chains | Main Menu → Chain Manager |
| `add_chain` | Select new chain type | Chain Manager → Add button |

### Chain and Processor Flow

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `chain_control` | Compound screen: chain graph + subscreen | Tap any chain strip |
| `control` | Parameter knobs for active processor | Default subscreen in chain_control |
| `chain_options` | Chain management: rename, move, remove | Long-press chain in chain_manager |
| `processor_options` | Per-processor: replace, move, randomize | Bold-press a processor node |
| `audio_out` | Audio output routing for this chain | chain_control → audio output node |
| `audio_in` | Audio input routing for this chain | chain_control → audio input node |
| `control_xy` | 2D touch pad for two parameters | From control screen — bold SW4 |

### Sound Selection

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `engine` | Engine category + list | add_chain → select type |
| `bank` | Bank list for selected engine | After engine selection |
| `preset` | Preset list for selected bank | After bank selection |

### MIDI Configuration

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `midi_config` | MIDI input/output port routing | chain_control → MIDI node |
| `midi_cc` | Per-chain CC routing (128 checkboxes) | chain MIDI settings |
| `midi_prog` | Program Change number selector | ZS3 options or PC binding |
| `midi_key_range` | Note range / keyboard split | chain MIDI settings |
| `midi_profile` | Load/switch MIDI profiles | Admin → MIDI |
| `midi_chan` | MIDI channel selector | chain options |

### State Management

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `snapshot` | Browse, load, save `.zss` files | Main Menu → Snapshots |
| `zs3` | Sub-snapshot list with PC info | Main Menu → ZS3 |
| `zs3_options` | Edit/delete/assign PC to a ZS3 | Bold-press a ZS3 entry |

### Sequencer / Pattern Editor

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `pated_notes` | Step sequencer note grid | Tap clip in launcher |
| `pated_cc` | CC automation alongside notes | Tab in pattern editor |

### Performance Tools

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `alsa_mixer` | ALSA hardware mixer controls | Main Menu → Soundcard Levels |
| `audio_player` | Play audio files | Main Menu → Audio Player |
| `midi_player` / `midi_recorder` | Record and play MIDI files | Main Menu → MIDI Player |
| `tempo` | BPM and tap tempo | Tap tempo in status bar |

### System / Admin

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `admin` | All system settings | Main Menu → Admin |
| `wifi` | Wi-Fi network scanner and connector | Admin → Wi-Fi Config |
| `bluetooth` | Bluetooth device management | Admin → Bluetooth |
| `brightness_config` | Display and LED brightness | Admin → Brightness |
| `cv_config` | Zynaptik CV/gate settings | Admin → CV Settings |
| `tts` | Text-to-speech accessibility | Admin → ZynVoice |
| `help` | Built-in HTML help viewer | Long-press Help button |

### Utility Screens

| Screen | Description | How to reach |
|--------|-------------|-------------|
| `option` | Generic single-choice list | Many contexts |
| `confirm` | Yes/No confirmation dialog | Destructive actions |
| `keyboard` | On-screen text input | Rename operations |
| `file_selector` | File browser | Snapshot import/export |
| `loading` | "Loading..." splash | During engine load |
| `dpm` | Digital peak meter (standalone) | Admin → Audio Levels |

---

## Navigation Patterns

### Generic Selector

Most screens use a scrollable list. The same interactions work everywhere:

| Input | Action |
|-------|--------|
| Encoder 3 rotate | Scroll list up/down |
| Encoder 3 push short | Select highlighted item |
| Encoder 3 push bold | Open options for highlighted item |
| Encoder 4 rotate | Adjust value (when in a parameter editor) |
| Touch tap | Select item |
| Touch long-press (~300ms) | Options for item (same as bold encoder) |
| SW2 short (Back) | Return to previous screen |
| SW1 short | Jump to Main Menu |

### Grid Selector

Used by `main_menu`, `add_chain`, `grid_sel`:

| Input | Action |
|-------|--------|
| Encoder 3 rotate | Move selection left/right |
| Encoder 4 rotate | Move selection up/down |
| Either encoder push | Activate selected button |
| Touch tap | Activate tapped button directly |

### Control Mode

Used by `control` and `control_xy`:

| Input | Action |
|-------|--------|
| Encoder 1 rotate | Adjust knob 1 (top-left parameter) |
| Encoder 2 rotate | Adjust knob 2 (top-right parameter) |
| Encoder 3 rotate | Adjust knob 3 (bottom-left) / scroll page list in select mode |
| Encoder 4 rotate | Adjust knob 4 (bottom-right) |
| Any encoder push short | Enter/exit select mode |
| Any encoder push bold | Enter MIDI learn for that knob |
| Touch tap knob | Select that knob |
| Touch long-press knob | MIDI learn for that knob |

---

## V5 Hardware Encoder Mapping

On V5 hardware, the four encoders have default functions that adapt per screen:

| Encoder | Mixer screen | Selector screens | Control screen |
|---------|-------------|-----------------|----------------|
| 1 (leftmost) | Select chain strip | — | Adjust knob 1 |
| 2 | Select phrase row | Scroll (context) | Adjust knob 2 |
| 3 | Volume fader | Scroll list | Adjust knob 3 / page select |
| 4 | Master volume | Value adjust | Adjust knob 4 |

---

## V5 Default Button Actions

| Switch | Short press | Bold press (300ms–2s) | Long press (>2s) |
|--------|-----------|-----------------------|-----------------|
| SW1 | MENU (main_menu) | SCREEN_ADMIN | POWER_OFF |
| SW2 | SCREEN_AUDIO_MIXER | SCREEN_ALSA_MIXER | ALL_SOUNDS_OFF |
| SW3 | CHAIN_CONTROL | BANK_PRESET | PRESET_FAV |
| SW4 | SCREEN_ZS3 | SCREEN_SNAPSHOT | — |

These are the defaults from `zynthian_envars_V5.sh`. Customizable via `ZYNTHIAN_WIRING_CUSTOM_SWITCH_NN__UI_SHORT/BOLD/LONG`.

---

## V5 Touch Keypad

The **V5 touch keypad** is a software button overlay that appears on the touchscreen when no physical V5 hardware buttons are present. It emulates all V5 button functions via touch, so VNC is not required for navigation.

Activate it in the Zynthian UI: **Admin → Touch Navigation → V5 keypad at left** (or at right).

### Screen Layout

With the keypad active, the 800×480 display is divided into three zones:

```
┌──────────┬──────────────────────────────────────────────────────────┐
│ OPT/     │ MIX/     │                                               │
│ ADMIN    │ LEVEL    │                                               │
├──────────┼──────────┤                                               │
│ CTRL/    │ ZS3/     │         Zynthian UI area                      │
│ PRESET   │ SHOT     │         (640 × 400 px)                        │
├──────────┼──────────┤                                               │
│ [metro]  │ PAD/     │                                               │
│          │ STEP     │                                               │
├──────────┼──────────┤                                               │
│ BACK/    │ SEL/     │                                               │
│ NO       │ YES      │                                               │
├──────────┼──────────┤                                               │
│ ALT      │ UP       │                                               │
├──────────┼──────────┤                                               │
│ LEFT     │ DOWN     ├────────┬──────┬──────┬──────┬──────┬──────┬──┤
│          │          │ RIGHT  │ ● REC│ ■ STP│ ▶ PLY│  F1  │  F2  │F3│F4│
└──────────┴──────────┴────────┴──────┴──────┴──────┴──────┴──────┴──┴──┘
 ←── 160px ──→ ←────────────── 640px ───────────────────────────────────→
```

- **Left side panel** — 160px wide × 480px tall — 2 columns × 6 rows of buttons
- **Bottom row** — 80px tall × 640px wide — 8 buttons (right of the side panel)
- **Zynthian UI area** — 640 × 400px — touch works normally here (tap, long-press, swipe)

### Button Reference

Button label format: **X/Y** means short tap = X action, bold hold (300ms) = Y action.

| Button | Short tap | Bold hold (300ms) | Long hold (>2s) |
|--------|-----------|-------------------|-----------------|
| **OPT/ADMIN** | Main Menu | Admin screen | Power off |
| **MIX/LEVEL** | Audio Mixer | ALSA Mixer (hardware levels) | All sounds off |
| **CTRL/PRESET** | Chain Control | Bank/Preset selection | Preset favorites |
| **ZS3/SHOT** | ZS3 list | Snapshots | — |
| **[metronome]** | Tempo | — | — |
| **PAD/STEP** | Pad Launcher | Pattern Editor | Arranger |
| **BACK/NO** | Back / cancel | — | — |
| **SEL/YES** | Select / confirm | Options for item | — |
| **ALT** | Toggle ALT mode | Help | — |
| **UP** | Navigate up | — | — |
| **LEFT** | Navigate left | — | — |
| **DOWN** | Navigate down | — | — |
| **RIGHT** | Navigate right | — | — |
| **REC (●)** | Record | — | — |
| **STOP (■)** | Stop | All notes off | All sounds off |
| **PLAY (▶)** | Play / pause | Audio file list | — |
| **F1** (ALT: F5) | Program Change 1 | Program Change 1 | — |
| **F2** (ALT: F6) | Program Change 2 | Program Change 2 | — |
| **F3** (ALT: F7) | Program Change 3 | Program Change 3 | — |
| **F4** (ALT: F8) | Program Change 4 | Program Change 4 | — |

**ALT mode:** Tap **ALT** to toggle. While active, F1–F4 relabel to F5–F8 and trigger Program Changes 5–8.

### Changing the Layout

| Setting | Where |
|---------|-------|
| V5 keypad at left | **Admin → Touch Navigation → V5 keypad at left** |
| V5 keypad at right | **Admin → Touch Navigation → V5 keypad at right** |
| Disable keypad | **Admin → Touch Navigation → None** |

The setting writes `ZYNTHIAN_UI_TOUCH_NAVIGATION2` in `/zynthian/config/zynthian_envars.sh` and takes effect after restart.

---

## Screen Navigation Stack

Zynthian maintains a linear screen history. Each screen transition pushes to the stack; Back pops. The stack resets to root on `show_screen_reset()` calls.

Typical depth when working:
```
mixer → chain_control → control → (CC learn inline)
      → chain_manager → add_chain → engine → bank → preset → control
```

In chain_control, the side chain panel toggles independently — it doesn't push a new screen, it expands/collapses within the current one.

---

## What's Next

- [Chains & Routing](chain-management.html) — add and configure chains
- [Control Screen](control-screen.html) — adjust synth parameters
- [ZS3 Subsnapshots](zs3-guide.html) — live performance recall
- [Admin & System](admin-guide.html) — WiFi, Bluetooth, updates

---

*Version: 2026-05-29 — derived from `zyngui/zynthian_gui_main_menu.py`, `zyngui/zynthian_gui_mixer.py`, `zyngui/zynthian_gui_chain_control.py`, `zyngui/zynthian_gui_touchkeypad_v5.py`, `zyngui/zynthian_gui_config.py`, `zynthian-sys/config/zynthian_envars_V5.sh`.*
