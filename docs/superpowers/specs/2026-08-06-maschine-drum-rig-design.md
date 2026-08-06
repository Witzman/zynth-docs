# Maschine MK2 Drum Rig — Design

**Date:** 2026-08-06
**Status:** approved, ready for implementation plan
**Scope:** sub-project 1 of 2. Sub-project 2 (2 Turing-machine voices controlled from SMC-PAD) gets its own spec after this one is verified on the Pi.

---

## Goal

Turn the Maschine MK2 into an 8-track × 16-step euclidean drum sequencer for live improvised techno, hosted entirely on Zynthian. Groups A–H are instruments, the 16 pads are steps of the selected group, encoders generate and shape patterns, F1–F8 mute tracks, Play/Restart drive transport.

---

## Feasibility summary

| Requirement | State before this work | Gap |
|---|---|---|
| 8 groups play simultaneously | daemon plays `current_page` only (`mikro.rs:533-631`) | real gap |
| Group select switches pad page | daemon: Group A–H = page | already works |
| enc 1–3 euclid params | only Shift+Group hit-count fill (`sequencer.rs:1`) | gap |
| enc 4/5 cutoff/res per group | encoders send plain CC ch 1 | needs routing |
| F1–F8 as mutes | buttons send CC, no mute logic | gap |
| Pads show steps + playhead | pad LEDs settable, no playhead render | gap |
| Play / Stop | Play starts, Erase stops; **no Stop button exists** on MK2 | remap |
| MK2 display | no MIDI path to LCD; raw HID half-working | out of scope |

Verified facts the design rests on:

- Daemon buttons and encoders already emit plain 7-bit CC on ch 1 (`midi_parse.rs:25`; `Message::RPN7` is this library's name for a 7-bit CC, not an RPN).
- Daemon OSC server on `127.0.0.1:42434` accepts per-pad RGB + brightness (`/maschine/pad i,i,f`) and per-button RGB + brightness (`/maschine/button/<name> i,f`) — `main.rs:609-665`. **No Rust changes are needed for LED feedback.**
- The MIDI-in LED path is weaker than OSC: NoteOn note < 16 only, global pad colour, no button LEDs (`main.rs:93-105`). OSC is therefore the LED transport.
- F1–F8 are the real names of the 8 buttons above the displays (`maschine.rs:25-32`). The transport row enum is `Restart, Grid, Play, Rec, Erase` — there is no `Stop`.
- Pad index 0 is bottom-left (commit `a42ff17`).
- FluidSynth in Zynthian is a shared multi-part process (`part_i` + `zmop_set_midi_chan_trans`, `zynthian_engine_fluidsynth.py:148-165`), so 8 chains cost one process.
- FluidSynth exposes `'filter cutoff'` (CC 74) and `'filter resonance'` (CC 71) as controllers (`zynthian_engine_fluidsynth.py:66-67`).
- zynseq provides everything the sequencer needs: `addNote`, `removeNote`, `setStepsPerBeat`, `setBeatsInPattern`, `toggleMute`, `isMuted`, `getPatternPlayhead`, `setPlayState` (`zynlibs/zynseq/zynseq.h`).
- Pattern step count is `beats × stepsPerBeat`, both integers (`pattern.cpp:655`).
- `setStepsPerBeat` rescales existing notes by `float(value)/m_nStepsPerBeat` (`pattern.cpp:670`).

**Difficulty:** Advanced. Touches a new Python ctrldev driver, zynseq internals, and an external Rust daemon.

---

## Architecture

```
MK2 hardware
  │ USB HID
MaschineMK2_linux daemon         ← unchanged code, runs in plain pad mode
  │ pads → NoteOn ch1 · F1-F8/Group/Play/Restart → CC ch1 · encoders → CC ch1
  ▼ ALSA
zynthian_ctrldev_maschine_mk2.py      ← NEW, the whole brain
  │  euclid math · group select · mute · transport · LED render
  ├──► zynseq   (8 sequences × one 16-step pattern, one scene)
  ├──► chain zctrls  ('filter cutoff' / 'filter resonance') of selected group
  └──► OSC 127.0.0.1:42434 → daemon → pad + button LEDs
                     │
zynseq playback ─────┴──► 8 chains, MIDI ch 1-8, shared fluidsynth, GM drum kit
```

### Chosen approach and rejected alternatives

**Chosen: Zynthian ctrldev driver.** The daemon degrades to a dumb control surface. All sequencing lives in zynseq.

Consequences that motivated the choice:

- Patterns, divisions and mutes save inside Zynthian snapshots — no separate persistence layer.
- The touchscreen pattern editor mirrors and edits the same data live.
- Zynthian owns clock and BPM, so no clock bridge and no drift for this rig.
- Follows the supported extension point — 30 drivers already exist in `zyngine/ctrldev/`, and `akai_apc_key25_mk2.py` (2824 lines) is a working reference for grid step-editing plus LED feedback plus mutes.

**Rejected: extend the Rust daemon to 8 parallel tracks.** Would mean reimplementing sequencing and per-track scheduling, patterns dying on daemon restart, a maintained fork, and a Zynthian pattern editor that cannot see any of it.

**Rejected: one drum engine with 8 notes.** Cheapest on CPU but cutoff/resonance would be global, which kills the enc 4/5 requirement.

---

## Drum voices

Eight chains, one per group, MIDI channels 1–8, FluidSynth with the stock GeneralUser GS GM drum kit. One shared fluidsynth process. Per-channel filter comes from the SF2's native CC 74 / CC 71 response — no extra filter plugin.

The GM kit is a PoC choice: it proves the rig end to end with zero sample sourcing. Swapping in a techno SFZ kit later changes only the chains, not the driver.

| Group | Channel | Instrument |
|---|---|---|
| A | 1 | kick |
| B | 2 | snare |
| C | 3 | clap |
| D | 4 | closed hat |
| E | 5 | open hat |
| F | 6 | rim / perc |
| G | 7 | tom |
| H | 8 | ride / crash |

Exact GM note numbers per instrument are set in the prepared snapshot, not hardcoded in the driver.

---

## Control map

| Control | Function |
|---|---|
| Group A–H | select group; pads and enc 1–5 follow selection |
| 16 pads | toggle a step of the selected group |
| Enc 1 | euclid hits, 0 to *step count* — regenerates the group's pattern |
| Enc 2 | division: 1/32, 1/16, 1/8, 1/16T, 1/8T (`setStepsPerBeat` 8, 4, 2, 6, 3) |
| Enc 3 | rotate pattern, 0 to *step count* − 1 |

*Step count* is 16 on straight divisions and 12 on the two triplet divisions, so the enc 1 and enc 3 ranges shrink on triplet groups. A hit count or rotation held over from a straight division is clamped to the new step count when the division changes.
| Enc 4 | `'filter cutoff'` zctrl of the selected group's chain |
| Enc 5 | `'filter resonance'` zctrl of the selected group's chain |
| F1–F8 | mute track 1–8, independent of the selected group (`toggleMute`) |
| Play | toggle Zynthian transport |
| Restart | all patterns to step 0 |
| Erase | reserved — clears the selected group's pattern |

Encoders 6–8 and the remaining buttons stay unassigned in this sub-project.

### Pattern authority

Enc 1–3 rewrite the selected group's 16 steps from the euclid parameters. Pad taps then toggle steps freely on top. The next enc 1–3 turn wipes those taps. No hidden per-step override state, no third LED state.

Because `setStepsPerBeat` rescales existing notes, a division change always regenerates the pattern from euclid rather than trying to preserve manual edits.

### LED language

| Surface | State |
|---|---|
| Pad | dim = empty step · bright = active step · white flash = playhead |
| Group A–H | lit = selected, dim = not selected |
| F1–F8 | lit = playing, dark = muted |

LED writes are diff-based over OSC — only pads and buttons whose state changed get a message. The repo has already hit a USB flood from unthrottled writes (commit `ffc8f2b`, display path), so throttling is a requirement, not an optimisation.

### Accepted consequences

1. **Triplet divisions have 12 steps, not 16.** Steps are `beats × stepsPerBeat` with both integers, so 16 steps at a triplet division is not representable. 1/16T (spb 6, beats 2) and 1/8T (spb 3, beats 4) both give 12 steps. Pads 13–16 go dark and inactive on those groups.
2. **Groups phase against each other.** 16 steps at 1/8 is two bars long; at 1/32 it is half a bar. Per-group division means groups drift out of alignment until they wrap. This is the intended polyrhythm behaviour.

---

## State ownership

| Data | Owner | Survives snapshot reload |
|---|---|---|
| 16 steps per group | zynseq pattern | yes |
| division | zynseq, read back via `getStepsPerBeat` | yes |
| hit count | derived — count active steps on group select | yes |
| rotation | driver RAM, defaults 0 | no |
| mute per group | zynseq `isMuted` | yes |
| selected group | driver RAM, defaults A | no |

The driver holds almost no state on purpose. After a snapshot reload, enc 1 and enc 2 resume from real values read out of zynseq. Only rotation resets — turning enc 3 then rotates from the pattern's current position.

### zynseq mapping

One scene. Eight sequences, one track each, one 16-step pattern each. Sequence *n* → MIDI channel *n* → drum chain *n*. Mutes address `(scene, phrase, sequence, track)`.

The PoC requires a **prepared snapshot** containing the 8 chains and the 8 sequences. The driver reads what exists and does not create it. Auto-provisioning of missing sequences is deferred out of the PoC.

### Pad preview

A ctrldev driver claims the MK2 input port exclusively, so pad notes never reach the chains by themselves. The driver therefore fires a preview hit of the selected group's instrument when a pad is tapped, so programming is audible.

---

## Failure modes and mitigations

| Risk | Effect | Mitigation |
|---|---|---|
| `dev_ids` string wrong | driver never loads, no error message | `[low]` — confirm the daemon's ALSA port name with `aconnect -l` on the Pi before writing the driver; list every plausible candidate string in `dev_ids` |
| Daemon in its internal sequencer mode | pads stop sending NoteOn, rig appears dead | rig requires plain pad mode; documented as a step. A Rust flag to lock the internal sequencer out is a later option |
| `setStepsPerBeat` note rescaling | edits mangled on division change | always regenerate the pattern from euclid after a division change |
| Playhead jitter | blink lands a frame late | LED refresh runs on the ctrldev refresh tick, not the audio clock. ~8 steps/sec at 130 BPM 16ths. Cosmetic only |
| Duplicate MIDI events | every press acts twice | SMC-PAD hit exactly this through a mirrored port. Check the MK2 for the same during Part 1 |
| Deploy gap | testing stale code | driver lives in `~/zynth/zynthian-ui/zyngine/ctrldev/` but must be copied to the Pi's `/zynthian/zynthian-ui/` each cycle. Every test part starts with the copy step |

---

## Testing

Pure functions — euclid-with-rotation, division mapping, hit-count derivation — are unit tested with pytest on WSL. No Pi and no hardware required.

Everything else is manual Pi verification, one part per sitting.

| Part | Adds | Verify |
|---|---|---|
| 1 | 8 chains + prepared snapshot + driver loads; Group A–H select; pads toggle steps; pattern LEDs | the touchscreen pattern editor shows the same steps you tapped |
| 2 | Play toggle, Restart, playhead LED | all 8 groups play together, playhead sweeps the pads |
| 3 | enc 1–3 euclid | density, division and rotation audible; triplet groups show 12 active steps |
| 4 | F1–F8 mutes | mute works regardless of selected group; LED matches mute state |
| 5 | enc 4/5 cutoff and resonance + pad preview hits | filter follows group selection; the touchscreen knobs move too |
| 6 | snapshot save and reload | patterns, divisions and mutes all return; enc 1/2 resume from real values |

Part 1 is the PoC — it proves daemon → driver → zynseq → sound end to end. If `dev_ids` resolves and pad taps land in zynseq, the rest is filling in the control map.

---

## Out of scope

- The MK2 LCD displays — no MIDI path exists to them, and the raw HID path is unresolved.
- Turing-machine voices and SMC-PAD control — sub-project 2, own spec.
- Auto-provisioning chains and sequences from the driver.
- A techno sample kit. GM drum kit for the PoC; kit swap is a chain change only.
- Making the daemon's CC map match NI factory MIDI mode (tracked separately in `MD/todo.md`).
