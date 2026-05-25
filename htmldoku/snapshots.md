# Snapshots

A snapshot is a complete save of the Zynthian state: all chains, engines, presets, MIDI assignments, mixer levels, and controller mappings. Loading a snapshot restores everything instantly.

---

## Saving a Snapshot

**From the touchscreen:** Press the Admin button → **Snapshots** → **Save Snapshot**. Enter a name and confirm.

**From webconf:** Open `http://zynthian.local` → **Snapshots**. A list of saved snapshots appears. Click **Save Current State** to overwrite, or **Save As** to create a new one.

Snapshots are stored as `.zss` files in `/zynthian/zynthian-my-data/snapshots/`. The file format is YAML-based and human-readable.

---

## Loading a Snapshot

**From the touchscreen:** Admin → **Snapshots** → select a snapshot from the list.

**From webconf:** Open **Snapshots** → click the snapshot name → **Load**.

On loading, Zynthian stops all current chains, starts the engines defined in the snapshot, and restores all settings.

---

## Auto-Load on Startup

The file `last_state.zss` in the snapshots directory is loaded automatically when Zynthian starts [`zynthian-ui/zyngui/zynthian_gui_admin.py`]. Saving a snapshot as `last_state` makes it the default startup state.

If `last_state.zss` does not exist (e.g. on a fresh install), Zynthian starts with an empty setup.

---

## Snapshot Library

Zynthian ships with example snapshots demonstrating common setups: a simple piano, a layered synth, an organ, etc. These are in `/zynthian/zynthian-data/snapshots/`. They are read-only; to customise one, load it and save a copy under a new name.

---

## Exporting and Importing Snapshots

**Export via SSH or SFTP:** Copy files from `/zynthian/zynthian-my-data/snapshots/` to your computer.

**Import:** Upload a `.zss` file to the same directory via SFTP or the webconf File Manager.

---

## What's in a Snapshot

A `.zss` file contains:

- Engine list per chain (engine name, bank, preset)
- MIDI channel assignments
- Audio routing (which chain goes to which output)
- Controller positions (knobs, faders)
- Mixer channel levels and mute/solo state
- Tempo and transport settings

It does not contain audio samples or soundfont data — those must be present on the device.

---

## What's Next

- [Synth Engines](synth-engines.md) — what engines can be saved in snapshots
- [Userguide](userguide.md) — chains and signal flow
- [Webconf Reference](webconf.md) — snapshot management in the web interface

---

*Version: 2026-05-25 — derived from `zynthian-ui/zyngui/zynthian_gui_admin.py`, `zynthian-ui/zyngine/zynthian_chain_manager.py`.*
