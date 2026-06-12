# Zynthian — To Do

Status: `[ ]` pending · `[~]` in progress · `[x]` done

Read this after `inwork.md` to see cross-cutting tasks and tutorial completion work.

---

## Active

- [~] **Complete Dub Techno Performance Loop tutorial**
  - [~] Test Part 1 on Pi — load snapshot `dub-techno-p1`, build patterns, verify playback
  - [ ] Draft Part 2 (pad + delay/reverb) — after Part 1 verified
  - [ ] Test Part 2 on Pi
  - [ ] Draft Part 3 (SMC-PAD mute control) — after Part 2 verified
  - [ ] Test Part 3 on Pi
  - [ ] Publish — run generator, commit, push, move to `done.md`
  - Plan: `~/zynth/docs/superpowers/plans/2026-06-04-dub-techno-loop.md`
  - Tutorial file: `~/zynth-docs/htmldoku/project-dub-techno-loop.md`
  - Snapshot on Pi: `/zynthian/zynthian-my-data/snapshots/000/dub-techno-p1.zss` (moved to 000/ bank — was at root, invisible in UI)

---

- [ ] **Test Dub Techno Live Rig — Maschine Pad Layer Part 1 on Pi**
  - [ ] Blocked: Maschine Step Sequencer Part 1 must pass first
  - [ ] Add pad chain on Ch2, verify 3-layer playback simultaneous
  - [ ] Test tempo drift over 8+ bars — document acceptable window
  - [ ] Test Shift+encoder B6 speed control (marked "under construction")
  - Tutorial file: `~/zynth-docs/htmldoku/project-dub-techno-maschine-pad.md`

---

- [~] **Debug and fix TOGGLE_SEQ — partially resolved, one issue remaining**

  **What was found and fixed (2026-06-04):**
  - SMC-PAD sends on **channel 6** (not 7 as tutorial stated — status byte `0x95` = ch6 1-indexed)
  - Master channel corrected: `ZYNTHIAN_MIDI_MASTER_CHANNEL=6` in `/zynthian/config/midi-profiles/default.sh`
  - All 16 mappings written to `ZYNTHIAN_MIDI_MASTER_NOTE_CUIA` with correct `\n` separators (not actual newlines)
  - `ZYNTHIAN_MIDI_MASTER_NOTE_CUIA` parser requires literal `\n` separators — actual newlines silently fail
  - SINCO Private port (card 4, port 0 = SINCO IN 1) mirrors all pad notes from SINCO Master (port 1 = SINCO IN 2)
  - Double-routing causes TOGGLE_SEQ to fire twice per press → double-toggle → no net change
  - Debounce added to `state_manager.py` on Pi: 50ms window per note (lines 836–840)
  - **MIDI reference page needs correction:** SMC-PAD channel is 6, master channel is 6, not 7

  **Remaining issue — TOGGLE_SEQ still not working after debounce:**
  - Debounce was added but TOGGLE_SEQ still didn't toggle with Launcher open
  - Possible causes not yet eliminated:
    1. Launcher has no patterns → `togglePlayState` succeeds but nothing visible
    2. `cuia_toggle_seq` uses flat sequence index — `TOGGLE_SEQ 0,0` passes `int(params[0])=0`, second param ignored
    3. `togglePlayState(bank, 0)` — bank may not have sequences set up in current dub-techno-p1 snapshot
    4. Debounce `_master_cuia_last` dict init may not have been applied correctly (check line ~234)
  - [ ] Load a snapshot with launcher patterns, open Launcher view, press Pad 1 — check if slot 0 highlights
  - [ ] Confirm `cuia_toggle_seq` receives correct params: add `print(cuia, params)` temporarily
  - [ ] Check if `togglePlayState(bank, 0)` requires a pre-existing sequence to have visible effect

  **Pi code state:**
  - `/zynthian/zynthian-ui/zyngine/zynthian_state_manager.py` — modified with debounce (not committed to git)
  - `/zynthian/config/midi-profiles/default.sh` — master channel = 6, 16 TOGGLE_SEQ mappings

  **Update MIDI Reference page:**
  - [x] SMC-PAD channel: change 7 → 6 everywhere — already done in current reference
  - [x] Master channel: change 7 → 6 everywhere — already done in current reference
  - [x] SINCO Private port double-routing: document as Conflict 10 — already present
  - [x] Maschine encoder/button MIDI type: updated RPN → standard CC (2026-06-06)
  - [x] Conflict 5 resolved — CC Learn now works for encoders and buttons (2026-06-06)

---

- [ ] **Verify Xboard 25 factory CC defaults**
  - [ ] Run `amidi -d -p hw:X,0,0` (X = Xboard card number from `aconnect -l`)
  - [ ] Turn each of 16 knobs, record CC number and channel
  - [ ] Check against SMC-PAD CCs (16/17/18/30/80/81/82/31) and common engine CCs
  - [ ] Update MIDI Reference Section 1 Xboard table with confirmed defaults
  - [ ] Remove `[low]` tag from Xboard knob row

- [ ] **Test Maschine MK2 Part 4 on Pi (web editor, MIDI IN)**
  - [x] SSH tunnel no longer needed — web editor at http://192.168.2.123:9000 (maschine-web.service)
  - [x] Confirm web editor loads — verified LAN access working
  - [x] Confirm pad LED changes on color set — fixed LED mapping (commit 1fb62eb), verified working
  - [x] Confirm maschine.json persists after restart — verified 2026-06-06 (pad note survives daemon restart)
  - [ ] Confirm MIDI Control IN drives pad LEDs

- [x] **Test Maschine MK2 Step Sequencer Part 1 on Pi** — verified (see tutorial)

- [ ] **Test Maschine MK2 Step Sequencer Part 2 on Pi (pages, per-step note/vel)**
  - [ ] Confirm Group A–H switch pages in sequencer mode
  - [ ] Confirm step selection (orange LED)
  - [ ] Confirm Encoder 1 = velocity, Encoder 2 = note offset
  - [ ] Blocked: Part 1 must pass first

- [ ] **Test Maschine MK2 Step Sequencer Part 4 on Pi (euclidean fill)**
  - [ ] Confirm Shift+Group D = 4 evenly-spaced hits on page 3
  - [ ] Verify exact step positions match table in tutorial
  - [ ] Blocked: Part 2 must pass first

- [x] **Test Maschine MK2 Step Sequencer Part 5 on Pi (MIDI clock sync)** — verified 2026-06-07, latest driver commits

---

## Backlog

- [ ] **Hardware patch bay for Cardinal standalone on Pi (future idea)**
  - Concept: physical breadboard patch bay → jumper wires trigger MIDI CC → Cardinal VCA matrix routes signals
  - Stack: Arduino Leonardo (USB MIDI) → matrix scan GPIO → Cardinal standalone on Pi (no Zynthian)
  - Cardinal patch: 2× VCO, sequencer, 2× ADSR, LFO, VCF, mixer, delay send, reverb send + 7×8 VCA matrix (56 VCAs)
  - Physical: 7 output pins (VCO1, VCO2, LFO, ADSR1, ADSR2, Seq CV, Seq Gate) + 8 input pins (VCF audio, VCF cutoff, VCA1 CV, VCA2 CV, VCO1 FM, VCO2 FM, delay send, reverb send)
  - CPU estimate: ~50% Pi 4 standalone — comfortable
  - Note: Cardinal can't dynamically create/destroy virtual cables via MIDI — VCA matrix (gain-controlled routing) is the implementation pattern
  - Parts needed: Arduino Leonardo (~€8), half-size breadboard, header pins, 10k pull-down resistors, jumper wires

- [ ] **Fix Maschine MK2 display (partially working — continue from investigation notes)**
  - Current state: `HEIGHT=64`, 2 reports (`byte3=0` then `byte3=32`), raw row-major → "readable but too big"
  - Investigation notes: `MD/display-investigation.md`
  - Next steps (in order): column offset test (buf[1]=64), bit reversal test, USB capture with usbmon
  - Source: `MaschineMK2_linux/src/devices/mk2/mikro.rs:431` (`send_display_bits`)
  - No commits needed for current state — it's the working baseline
