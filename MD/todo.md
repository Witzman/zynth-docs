# Zynthian — To Do

Status: `[ ]` pending · `[~]` in progress · `[x]` done

Read this after `inwork.md` to see cross-cutting tasks and tutorial completion work.

---

## Active

- [~] **Maschine MK2 Drum Rig — implementation plan** (2026-08-07, resume at task 10)
  - Read first: `~/zynth/zynthian-ui/.superpowers/sdd/2026-08-06-maschine-drum-rig/progress.md` (ledger), then the plan and spec in `docs/superpowers/`
  - [x] Task 7 — hardware-verified 2026-08-07 after four bug fixes plus the `external_pad_leds` daemon flag; Play button LED added on request
  - [x] Task 8 — euclid encoders — hardware-verified. Enc 1 = hits, enc 2 = rotation, enc 3 = division (all five divisions, triplets included)
  - [x] Task 9 — hardware-verified. F1-F8 mutes (selection-independent), pad preview via `libseq.playNote`, Erase, enc 8 = volume, group buttons coloured to match their pads
  - [x] Group button colour now matches its pads — report 0x81 group buttons are full RGB, 3 contiguous bytes each (starts 1, 7, 13, 22, 25, 34, 37, 46)
  - [x] **`2fc6a837` hardware-tested 2026-08-08** — five defects found, all fixed and re-verified. Driver now at `8c4e9f70`:
    - `21912769` sample switching listened on CC 48/47 (page buttons, which the daemon swallows); the arrows beside the display send CC 5/6
    - `1caa427c` polyrhythm: LOOP mode was forced only in `init()`, and a snapshot load restores LOOPALL — a short LOOPALL sequence goes silent until the next bar sync. Re-forced on write and transport start. 12 vs 16 realigns every **3** bars, not 4
    - `07b8d41b` encoder 7 (release) dropped — inaudible, same unipolar-modulator dead end as the filter
    - `48361eba` length changes now preserve hand-edited steps (shrink keeps what fits; growing back does not restore what was dropped)
    - `8c4e9f70` F1-F8 mute the mixer strip, not the zynseq track — zynseq's format has no mute field, so mutes never survived a save; plus LEDs now repaint on `SS_LOAD_SNAPSHOT`
    - Phantom extra drum sounds on every pad tap were a stale manual `jack_connect` to `dev3_in`, not code. `zynautoconnect` only removes routes it made itself, and jackd outlives a zynthian restart
  - [~] Task 10 — snapshot round-trip and tracking files **done**; **tutorial page remains** (user deferred it)
  - [x] Pushed 2026-08-08 — `zynthian-ui` vangelis `8c4e9f70`, `zynth-docs` master
  - [ ] Pattern length is quantised to whole beats and cannot be otherwise (`getLength() = beats * PPQN`, no `setSequenceLength` in the installed API). Step counts 1, 5, 7, 11, 13 are unreachable; a 1/4 division (spb 1) would unlock them at quarter-note steps — deferred by the user
  - [ ] Test snapshot `000/015-212121.zss` on the Pi — delete it or keep it as the round-trip fixture
  - [ ] Display: map the row order by drawing single rows one at a time (y=0,1,2,3,8,9). Screens are 512x64 and a text row at y=0 works; rows past ~8 drop content. Full notes in `MD/display-investigation.md`. Do not build a multi-row layout first
  - [ ] Display: wire the working top text row to the driver — group labels under F1-F8 (deliverable now)
  - [ ] Per-group kit switching across the 42 drum-machine SFZ kits in `/zynthian/zynthian-data/soundfonts/sfz/Drum Machines/` — bigger sonic win than any CC
  - [ ] Unset `ZYNTHIAN_LOG_LEVEL` on the Pi once task 10 is done: `systemctl unset-environment ZYNTHIAN_LOG_LEVEL`
  - [x] Cold-boot ordering race — survived a real cold boot 2026-08-07: alias present, `Pads MIDI → ZynMidiRouter:dev2_in` bound. One sample only; still worth `After=maschine-mk2.service` if it ever recurs
  - [ ] Filter control needs an LV2 filter in each chain — FluidSynth's CC 74/71 are unipolar and `FluidDrums.sf2` ships wide open, so they can never be audible
  - [ ] `light_buf2` bytes 17-31 unverified (17-24 Scene/Pattern/Pad Mode row, 25-31 master section); `Shift`, `Erase`, `Rec`, `Grid`, `Stepleft/right`, `Restart` in `light_buf3` unverified. Map with `/maschine/rawled` if any needs to light
  - [ ] Re-run `tools/patch-autoconnect-maschine.py` after any Zynthian update, or the driver silently stops binding
  - [ ] Sub-project 2 — two Turing-machine voices on the SMC-PAD: needs its own spec and plan

  **Hard rules learned 2026-08-07 — do not relearn:**
  - Any zynseq access added to the driver MUST take `self.lock`. libzynseq is not thread-safe and unsynchronised access from the poll thread segfaulted the whole UI (exit 139) ~95s into a jam
  - Never drive anything step-rate-sensitive from `SS_SEQ_PROGRESS` — it is 5 Hz (`slow_thread_task`, 0.2s sleep) and aliases against the step rate
  - `TOGGLE_PLAY` is not a sequencer transport; it resolves to `cuia_toggle_audio_play()`. Use `setPlayState` directly
  - `external_pad_leds: true` must stay in the daemon's `maschine.json` or pad colours die on first touch

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

- [ ] **MaschineMK2_linux — MIDI-mode compatibility (from CE analysis, 2026-08-06)**
  - Source of truth: `htmldoku/project-midi-reference.md` §"Maschine MK2 — factory MIDI mode"
  - Option A: make daemon's CC map match NI factory MIDI mode (Play=108, Rec=109, Group A–D=80–83, knobs CC 14–21 / 22–29, F1–F8 CC 46–53 / 54–61). Would make stock MK2 DAW templates work unchanged; breaks current tutorials that cite CC 1–14 / 24–48
  - Option B: leave map as-is, add a selectable "NI compat" profile in `maschine.json`
  - Related: `MIDI Control` IN currently takes NoteOn 0–15 for pad LEDs. NI convention is HSB triplets sharing one note across ch 1/2/3. Conversion reference: `toHSB()` in `CE/Template Support Files/Ableton Live 9/Maschine_Mk2/MIDI_Map.py`
  - Confirmed dead end: Controller Editor exposes no display access — no MIDI path to the LCD

- [ ] **SMC-PAD is reflashed to NiFox Koala preset — pads now ch 10 (confirmed 2026-08-06)**
  - Every ch-6 assumption in the rig is stale: master channel, drum chains, `DRUM_CHAN`, ctrldev ZYNPOT CCs
  - Full analysis: `htmldoku/project-midi-reference.md` Conflict 11
  - Fix is one filter rule — `MAP CH#9 => CH#5` in webconf → Interface → MIDI Options → Midi filter rules (channels 0-indexed). Bank A pad notes are still 36–51, so nothing else needs touching
  - [ ] Verify on Pi: `amidi -d -p hw:X,0,0`, hit pad 1 → expect `99 24 vv`
  - [ ] Capture the 8 encoder CCs — NiFox moved them to 30–37, ctrldev still listens on CC 16/17/18/30
  - [ ] Apply filter rule, retest drum chain + Launcher
  - [ ] Decide: keep NiFox permanently (filter rule stays) or reflash factory preset 1 when using Zynthian
  - Unresolved: which of the 5 presets sits in which device slot; what sends notes 111–126 ch 1

- [ ] **Fix Maschine MK2 display (partially working — continue from investigation notes)**
  - Current state: `HEIGHT=64`, 2 reports (`byte3=0` then `byte3=32`), raw row-major → "readable but too big"
  - Investigation notes: `MD/display-investigation.md`
  - Next steps (in order): column offset test (buf[1]=64), bit reversal test, USB capture with usbmon
  - Source: `MaschineMK2_linux/src/devices/mk2/mikro.rs:431` (`send_display_bits`)
  - No commits needed for current state — it's the working baseline
