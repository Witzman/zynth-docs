# Snapshots

A snapshot is a complete save of the Zynthian state: all chains, engines, presets, MIDI assignments, mixer levels, and controller mappings. Loading a snapshot restores everything instantly.

---

## Saving a Snapshot

**From the touchscreen:** Tap **OPT/ADMIN** (short) → **Snapshots**. Navigate into a bank (e.g. **000**). Tap **Save as new snapshot**, enter a name, confirm.

**From webconf:** Open `http://zynthian.local` → **Snapshots**. Navigate into a bank folder first, then type a name in the **Name:** field and tap the checkmark icon.

> **Bank subdirectory required.** Snapshots saved to the root of `/zynthian/zynthian-my-data/snapshots/` are invisible in the Zynthian UI snapshot list. Always save into a bank subfolder — the default is `000`. To move a misplaced snapshot via SSH:
> ```bash
> mv /zynthian/zynthian-my-data/snapshots/name.zss /zynthian/zynthian-my-data/snapshots/000/
> ```

Snapshots are stored as `.zss` files under `/zynthian/zynthian-my-data/snapshots/<bank>/`. The file format is YAML-based and human-readable.

---

## Loading a Snapshot

**From the touchscreen:** Tap **ZS3/SHOT** (bold hold, 300ms) to open the Snapshots screen. Navigate into a bank, tap the snapshot name to load it.

**Alternatively:** Tap **OPT/ADMIN** (short) → **Snapshots** → navigate into bank → tap snapshot name.

**From webconf:** Open `http://zynthian.local` → **Snapshots** → click the snapshot name → **Load**.

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
