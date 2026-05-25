# ZS3 Subsnapshots

A **ZS3** (Zynthian Sub-Snapshot) is a partial state save. Unlike a full snapshot (which captures everything), a ZS3 can capture specific chains and parameters — and it can be recalled instantly via MIDI Program Change during a live performance.

---

## ZS3 vs Snapshot

| Feature | Snapshot | ZS3 |
|---------|----------|-----|
| Saves all chains | Yes | No — selective |
| Saves engine/preset | Yes | Optional |
| Saves parameters | Yes | Yes |
| Saves mixer levels | Yes | Optional |
| Recall method | Manual (screen/file) | Program Change or screen |
| Use case | Between songs | Between sections within a song |

---

## ZS3 Screen

Access: Main Menu → **ZS3** (or SW3 on V5 default mapping).

The list shows:

1. **Save as new ZS3** — captures current state as a new ZS3 entry.
2. **Default state** (`zs3-0`) — the baseline state; always present.
3. **Saved ZS3s** — all named ZS3 entries you have created.

Each entry shows its title and (if linked to Program Change) the PC number and channel.

**Select (short press):** load that ZS3 immediately — applies its saved parameters.

**Bold press:** open ZS3 options for that entry.

Source: [`zyngui/zynthian_gui_zs3.py`](../zynthian-ui/zyngui/zynthian_gui_zs3.py)

---

## Saving a ZS3

1. Set up the state you want to capture (preset, volume, effects, etc.).
2. ZS3 screen → **Save as new ZS3**.
3. A keyboard screen appears — type a name (e.g. "Verse", "Chorus").
4. The new ZS3 appears in the list.

To **update** an existing ZS3 with the current state: bold-press the entry → **Overwrite**.

---

## ZS3 Options

Bold-press any ZS3 entry to open options:

| Option | What it does |
|--------|-------------|
| Restore options | Choose which chains/processors are restored when loading |
| Overwrite | Replace this ZS3 with the current state |
| Rename | Change the title |
| Delete | Remove this ZS3 |
| Program Change Number | Assign a MIDI PC number (0–127) to recall this ZS3 |
| Program Change Channel | Restrict PC recall to a specific MIDI channel (1–16 or Any) |

Source: [`zyngui/zynthian_gui_zs3_options.py`](../zynthian-ui/zyngui/zynthian_gui_zs3_options.py)

---

## Restore Options

By default, loading a ZS3 restores all parameters it captured. The **Restore options** submenu lets you toggle which chains and processors are actually applied on recall.

Each chain and processor shows a checkbox: ☑ = will be restored, ☐ = skip on recall.

**Use case:** you have a ZS3 that changes organ drawbars but you don't want it to reset the piano preset. Uncheck the piano chain — only the organ parameters apply.

A **Toggle All Mixer** option toggles all audio mixer processors at once.

---

## Linking ZS3 to MIDI Program Change

This is the key live performance feature — your foot switch or controller can jump to any ZS3 without touching the screen.

### Method A: Assign via ZS3 Options

1. ZS3 screen → bold-press the ZS3 you want to link.
2. Options → **Program Change Number** → select 0–127.
3. Optionally: **Program Change Channel** → select a specific channel, or leave as "Any".
4. The ZS3 list entry now shows "→ PRG#N" (or "CH#N:PRG#N" for a specific channel).

From now on, when Zynthian receives PC#N on that channel, this ZS3 loads automatically.

### Method B: MIDI Learn

1. On the ZS3 screen, press the MIDI learn button (encoder select long-press, or the learn soft button).
2. Screen shows **"Waiting for MIDI Program Change..."**
3. Send a Program Change from your controller.
4. The received PC number is assigned to the currently selected ZS3.

The learn mode is cancelled when you leave the ZS3 screen.

---

## ZS3 ID Format

Internally, ZS3 entries use these ID formats:

| ID format | Meaning |
|-----------|---------|
| `zs3-0` | Default state (always present) |
| `zs3-N` | Named ZS3 without PC assignment |
| `*/prog` | Any channel, Program Change = prog |
| `chan/prog` | Specific channel `chan`, Program Change = prog |

---

## Example: Live Song Arrangement

Song: "Blue Sky" — three sections.

1. Set up the full live rig (piano chain + organ chain + drum chain).
2. **Save a snapshot** as "Blue_Sky" for the whole song.
3. Adjust organ drawbars for the verse — save as ZS3 "Verse" → PC#0.
4. Pull out the chorus drawbars — save as ZS3 "Chorus" → PC#1.
5. Reduce organ volume for the bridge — save as ZS3 "Bridge" → PC#2.

During the performance, a foot switch sends PC#0 (verse), PC#1 (chorus), PC#2 (bridge) to instantly morph the rig between sections — no hands needed.

---

## Example: Foot Switch Recall

Hardware: sustain pedal or foot switch sending Program Change on channel 16.

1. ZS3 screen → bold-press target ZS3 → Program Change Number = 0, Channel = 16.
2. Press sustain pedal — ZS3 loads.
3. To step through multiple ZS3s sequentially, assign consecutive PC numbers and program your foot switch to increment.

---

## What's Next

- [Snapshots](snapshots.html) — full-state saves between songs
- [Control Screen](control-screen.html) — save parameters into a ZS3
- [MIDI Controllers](midi.html) — configure your foot switch or controller

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_zs3.py`, `zyngui/zynthian_gui_zs3_options.py`.*
