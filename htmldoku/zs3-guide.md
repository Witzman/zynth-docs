# ZS3 Subsnapshots

A **ZS3** (Zynthian Sub-Snapshot) is a partial state save that can be recalled instantly during a live performance via MIDI Program Change. Unlike a full snapshot (which captures everything and takes a moment to load), a ZS3 applies selectively and instantly.

---

## ZS3 vs Snapshot

| Feature | Snapshot (`.zss`) | ZS3 |
|---------|-------------------|-----|
| Saves all chains | Yes | No — selective per chain/processor |
| Saves engine / preset | Yes | Optional (toggle per chain) |
| Saves synth parameters | Yes | Yes |
| Saves mixer levels | Yes | Optional (toggle) |
| Saves MIDI routing | Yes | No |
| Recall method | Manual (screen or file load) | MIDI Program Change or screen tap |
| Load time | 1–5s (engine restart) | Instant (<100ms) |
| Typical use | Between songs | Between sections within a song |
| Max per session | One per file | Dozens per snapshot |
| Stored in | Separate `.zss` file in a bank subdir | Inside the loaded `.zss` file |

---

## ZS3 Screen

**From the touch keypad:** Tap **ZS3/SHOT** (short) — opens the ZS3 list directly.

**Alternatively:** Tap **OPT/ADMIN** (short) → **ZS3**.

**V5 hardware:** SW4 short.

> **ZS3/SHOT bold hold (300ms)** opens the Snapshots screen (`.zss` files) instead — not the ZS3 list.

The list shows three sections:

1. **Save as new ZS3** — captures the current complete state as a new ZS3 entry.
2. **Default state** (`zs3-0`) — always present; the baseline state. Reloading this restores whatever was saved as the default.
3. **Saved ZS3s** — all named ZS3 entries you have created, with PC number / channel shown if assigned.

**Entry format in the list:**
- `Title` — named ZS3, no PC assigned
- `Title → PRG#N` — assigned to Program Change N on any channel
- `Title → CH#C:PRG#N` — assigned to Program Change N on channel C

### Screen Interactions

| Action | Result |
|--------|--------|
| Short press (tap) | Load that ZS3 immediately |
| Bold press | Open ZS3 options for that entry |
| MIDI learn button (encoder long-press) | Enter waiting mode — next PC received assigns to selected ZS3 |

Source: [`zyngui/zynthian_gui_zs3.py`](../zynthian-ui/zyngui/zynthian_gui_zs3.py)

---

## Saving a ZS3

**Save new:** ZS3 screen → **Save as new ZS3** → keyboard screen appears → type a name → confirm.

The new ZS3 captures the current state of all chains and processors. By default, all of them are marked for restore.

**Update existing:** bold-press the entry → ZS3 options → **Overwrite**. Replaces the saved state with the current state, keeping the same name and PC assignment.

---

## ZS3 Options

Bold-press any ZS3 entry to access its options:

| Option | Detail |
|--------|--------|
| **Restore options** | Choose which chains/processors are included when loading this ZS3 |
| **Overwrite** | Replace this ZS3 with the current state |
| **Rename** | Change the title (keyboard screen) |
| **Delete** | Remove this ZS3 (with confirmation) |
| **Program Change Number** | Assign MIDI PC 0–127 (or None) to trigger this ZS3 |
| **Program Change Channel** | Restrict PC recall to channel 1–16 (or Any) |

**Note:** `zs3-0` (Default state) only shows Overwrite — it cannot be renamed, deleted, or assigned a PC.

Source: [`zyngui/zynthian_gui_zs3_options.py`](../zynthian-ui/zyngui/zynthian_gui_zs3_options.py)

---

## Restore Options

The **Restore options** submenu controls which parts of the ZS3 are actually applied when loaded.

### The Options Screen

Shows a tree: chains at the top level, processors indented below. Each entry has a checkbox:

| Checkbox | State |
|---------|-------|
| ☑ | This item WILL be restored when the ZS3 loads |
| ☐ | This item will be SKIPPED on load |

**Toggle a single item:** tap it (short press).

**Toggle all mixer items at once:** the **Toggle All Mixer** entry at the top — this controls all audio mixer (`MI` and `MR`) processors simultaneously. Useful for quickly including or excluding all mixer level changes.

### Practical Use of Restore Options

| Scenario | What to do |
|----------|------------|
| ZS3 changes organ drawbars only, not piano preset | Uncheck the piano chain |
| ZS3 adjusts mixer levels but not engine params | Uncheck synth processor, check mixer processor |
| ZS3 is a full state change | Leave all checked (default) |
| ZS3 changes one LFO rate and nothing else | Uncheck all except that one processor |

---

## Program Change Assignment

This is the key live performance feature. Your foot switch or MIDI controller can jump to any ZS3 without touching the screen.

### Method A: Assign via ZS3 Options

1. Bold-press the ZS3 you want to link.
2. ZS3 options → **Program Change Number** → select 0–127 (or None to remove).
3. Optionally: **Program Change Channel** → select 1–16 (or Any).
4. The ZS3 list now shows "→ PRG#N" after the title.

### Method B: MIDI Learn

1. On the ZS3 screen, navigate to the ZS3 you want to link.
2. Long-press encoder 3 (or use the learn button) — the screen shows **"Waiting for MIDI Program Change..."**
3. Send a Program Change from your controller.
4. The PC number is captured and assigned to the selected ZS3.
5. Press Back to cancel waiting mode without assigning.

---

## ZS3 ID Format

Internally, ZS3 entries use these ID formats stored in `state_manager.zs3`:

| ID format | Meaning |
|-----------|---------|
| `zs3-0` | Default state — always present, cannot be deleted |
| `zs3-N` (e.g. `zs3-1`) | Named ZS3 without PC assignment |
| `*/prog` | Any channel, Program Change = `prog` (0-indexed) |
| `chan/prog` | Specific channel `chan` (0-indexed), Program Change = `prog` |

When you assign PC#1 on Any channel, the ID becomes `*/1`. When you assign PC#1 on channel 16, the ID becomes `15/1` (channels are stored 0-indexed).

---

## Admin Setting: Program Change for ZS3

**Admin → MIDI → Program Change for ZS3** controls how Zynthian handles incoming PC messages globally:

| Setting | Behavior |
|---------|---------|
| ☑ Enabled | PC messages recall ZS3 states |
| ☐ Disabled | PC messages switch presets in the active engine |

When ZS3 mode is on, all PC messages go to the ZS3 system. When off, they go to the chain's engine. Choose based on your performance workflow.

Stored as env var: `ZYNTHIAN_MIDI_PROG_CHANGE_ZS3` (0 or 1).

---

## Example: Live Song Arrangement

Song "Blue Sky" — intro, verse, chorus, bridge, outro — all on one FluidSynth + setBfree + drums rig.

**Setup (once):**

1. Build the complete rig: piano chain (ch 1), organ chain (ch 2), drums chain (ch 10).
2. Save a snapshot: "Blue_Sky.zss" — this is loaded at the start of the set.
3. **Intro:** organ drawbars at soft setting + piano quieter → save ZS3 "Intro" → PC#0.
4. **Verse:** organ medium + piano normal → save ZS3 "Verse" → PC#1.
5. **Chorus:** organ all drawbars out + piano louder → save ZS3 "Chorus" → PC#2.
6. **Bridge:** mute organ, piano with reverb → save ZS3 "Bridge" → PC#3.
7. Set restore options on "Verse" to exclude the drums chain (drums don't change).
8. Enable: Admin → MIDI → Program Change for ZS3 = ☑.

**During performance:** foot switch sends PC#0 → PC#1 → PC#2 → PC#3 → PC#1 → PC#2, morphing the rig between sections with no hands on the screen.

---

## Example: Foot Switch Recall with Channel Filtering

Hardware: M-Audio sustain pedal programmed to send PC#0 on channel 16.

1. ZS3 "Main Sound" → bold press → Program Change Number = 0, Channel = 16.
2. The ZS3 only triggers on PC#0 from channel 16. Other PC messages on other channels are not affected — they can still control presets in individual chains.

---

## What's Next

- [Snapshots](snapshots.html) — full-state saves between songs
- [Control Screen](control-screen.html) — save parameters into a ZS3
- [MIDI Controllers](midi.html) — configure your foot switch or controller
- [MIDI CC Learning](midi-cc-learn.html) — bind additional controls

---

*Version: 2026-05-25 — derived from `zyngui/zynthian_gui_zs3.py`, `zyngui/zynthian_gui_zs3_options.py`, `zyngui/zynthian_gui_admin.py`.*
