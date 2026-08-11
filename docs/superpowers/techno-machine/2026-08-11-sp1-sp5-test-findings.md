# SP1 + SP5 Hardware Test — Findings

**Date:** 2026-08-11
**Tested by:** the owner, at the panel, with the driver and daemon deployed
**Build under test:** `zynthian-ui` vangelis `78c0659a`, `MaschineMK2_linux` main `39c4503`

---

## Passed

| Check | Result |
|---|---|
| Five modes latch, exactly one LED lit | PASS |
| VOLUME (MIXER) and AUTO (FILTER) as modes | PASS — both buttons did nothing before today |
| Pressing a lit mode returns to CONTROL | PASS |
| **DL/DR paging** — `LEVEL 1/3 → REVERB 2/3 → DELAY 3/3` | **PASS — the CC 47/48 correction is confirmed on hardware** |
| Ring wraps forward and backward | PASS |
| TL/TR do nothing (deliberately unbound) | PASS |
| Spread page addresses eight channels — encoder 3 moves C, encoder 6 moves F | PASS |
| FILTER greys the five drum columns; a greyed knob is silently dead | PASS |
| Page memory per (mode, kind) — leaving a voice and returning keeps its page | PASS |
| `1/4` division reachable, appended last | PASS |
| **SP5 stuck-note gate** — long note on the last step, four-bar loop | **PASS — no stuck note** |
| Long notes audible — GATE min → stabs, GATE max → sustained | PASS |
| **T4** — RANDOM > 0 with long notes, pattern rewriting every bar | **PASS — no stranded notes** |
| Voice DENSITY thins the pattern, rests stay in place | PASS |
| DENSITY 0 silences the channel **and draws its tab dashed** | PASS |
| Peak meters move with audio; a silent channel's bar does not flicker | PASS — confirms the mixer-API fix and the meter quantisation |
| Drum sample stepping on ML/MR | PASS |

**Note on the stuck-note gate:** it proves the *clamp* works. It does not prove
a note may safely cross the loop point, because with the clamp in place none
ever does. Removing the clamp still requires a deliberate experiment with an
unclamped build.

---

## Defects found

### 1. Play chance is invisible and unrecoverable after a snapshot load — **the silent-channel law, violated by a stale-state path**

**Symptom:** channels G (LEAD) and H (PADS) were completely silent after loading
snapshot `016-techno_maschine`, while the pads showed 14 lit steps, the playhead
ran across them, the mixer showed level 100, the tab drew **solid**, and a pad
tap previewed the note correctly through the engine.

**Cause:** `setPlayChance` is a zynseq **pattern** property and is saved inside
the snapshot's `zynseq_riff_b64`. The driver does **not** persist chance in its
own state (`get_state` saves `register`, `ring`, `length`, `random`, `gate`,
`octave`, `range`, `velo`, `density` — not `chance`) and does not read it back
from zynseq on load. So after a snapshot written with chance 0, zynseq holds 0
while the driver's state dict holds its default of 100.

**Why it matters more than the silence:** the dashed-tab indicator reads the
driver's state dict, so it reports the channel as healthy. The one mechanism
this instrument has for explaining silence was itself lying, which is exactly
the failure the dashed tab exists to prevent.

**Workaround used during the test:** turn the channel's CHANCE to any *different*
value and back — `apply()` early-returns when the value is unchanged, so only a
real change writes through to zynseq.

**Fix (not yet built):** on `SS_LOAD_SNAPSHOT`, read each channel's real
`getPlayChance()` out of zynseq into the state dict, so the surface and the
sequencer agree. Same argument applies to swing, which is also a per-pattern
zynseq property the driver asserts rather than reads.

### 2. Snapshot `016-techno_maschine` ships with LEAD and PADS muted

`chan_06` and `chan_07` both carry `mute: 1` in the snapshot's `zs3-0` mixer
state; every other channel has `mute: None`. Unrelated to this build — those two
channels have been silent on every load of that snapshot. Unmuting works from
the F row (F7, F8). **The snapshot should be re-saved unmuted.**

### 3. Long notes plus repeated pitches cut each other off; swing exposes it

**Symptom:** on a voice at `1/4` with GATE at maximum, raising SWING silenced the
sequence entirely. Returning SWING to minimum restored it.

**Discriminating test:** with GATE low, swing at `1/4` plays fine. So **swing is
not the defect.**

**Cause:** with GATE 800 a note is eight steps long on a sixteen-step pattern, so
each pitch is still sounding when it comes round again. `track.cpp:186-192` adds
swing as a fraction of a step to the event offset, and the note-off is scheduled
at `(offset + duration) × clocksPerStep`. The shift is enough for a previous
instance's note-off to land after the new note-on and cut it.

**Owner's ruling:** the swing behaviour at `1/4` is musically interesting and
stays as it is. The interaction is recorded rather than fixed.

**If it is ever fixed**, the shape would be clamping a note's duration to the
gap before the next note of the same pitch — legato rather than overlap — which
is a second clamp alongside SP5's loop-point one.

### 4. A very short GATE on a slow-attack patch is inaudible

LEAD appeared to have "only a few notes" until GATE was raised. At the bottom of
its range a note is 0.05 steps, which on a patch with any attack at all never
develops. Combined with the coarser encoder resolution the widened range
introduced, the bottom of GATE is now easier to land on by accident. Not a bug;
worth knowing, and an argument for a coarse/fine split on that knob if it
annoys in practice.

### 5. Voice preset stepping on MR — unresolved

Stepping presets on channel F appeared to do nothing audible. The journal shows
`set_preset: Preset selected: BA 002 ng (9)` → `BA 003 ng (10)` → `BA 002 ng (9)`,
so the mechanism fired. Two candidate explanations, neither confirmed: the JC303
bass patches involved sound near-identical, and the `PRESET` column lives on the
CONTROL page while the test was run from STEP. **Re-test on LEAD or PADS from the
CONTROL page.**

---

## Still untested

- SOLO gestures (the oldest unverified behaviour in the project)
- Snapshot round trip of the new division and gate
- Generated CONTROL pages on a voice (`EXTRA` pages built from plugin ports)
- ALL-page generated reverb/delay pages
- The twenty-minute stability jam
