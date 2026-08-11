# Zynthian — To Do

Status: `[ ]` pending · `[~]` in progress · `[x]` done

Read this after `inwork.md` to see cross-cutting tasks and tutorial completion work.

---

## Active

- [~] **Techno Machine pass two — SP1 mode & page framework** (2026-08-11, **resume at Task 10**)
  - Read first: `docs/superpowers/plans/2026-08-11-techno-machine-pass-two-sp1.md`, then its spec `docs/superpowers/specs/2026-08-11-techno-machine-pass-two-design.md`
  - Five latched modes (CONTROL · STEP · ALL · MIXER on VOLUME · FILTER on AUTO), each a ring of parameter pages stepped with DL/DR
  - [x] Task 1 — page shapes, descriptors, `PAGE_RINGS` keyed `(mode, kind)` — `22d217a3`
  - [x] Task 2 — `columns()` renders all three shapes; `page_label()`, `quantise_frac()` — `287a3aa5`
  - [x] Task 3 — generated pages built from a plugin's own ports, with a port filter — `ad20b000`
  - [x] Task 4 — page-indicator row; **found and fixed a shadowing bug in shipped `screen_packets`** (a loop local named `label` clobbered the new parameter) — `7fc9372a`
  - [x] Task 5 — `self.mode` + `self.page_idx`, snapshot round-trip, voices gain `chance` — `3928cbf6`
  - [x] Task 6 — VOLUME/AUTO bound as modes, DL/DR paging, ML/MR sound stepping, five-mode LEDs — `6ebb20fb`
  - [x] Task 7 — `_encoder_column` dispatches on shape; `COLUMN_VERBS` retired — `00c59b7a`
  - [x] Task 8 — spread columns, page label, peak meters off the real `zynmixer.DPM` struct — `06cc3fc8`
  - [x] Task 9 — generated ring cache, invalidated on snapshot/kit/preset — `1cbe5f95`
  - [x] **Task 10 — daemon patch** — `MaschineMK2_linux` `39c4503`. Three RPN7 arms emitting SHIFT 49, SWING 50, VOLUME 51 after `page_left`. `cargo build --release` clean, no new warnings. The `set_mod` block above the match is untouched — SHIFT is still a live internal modifier gating PAD MODE and the B6 encoder
  - [x] **Task 11 — the G4 runbook** — `zynth-docs` `a181987`, at `docs/superpowers/techno-machine/2026-08-11-gate-g4-runbook.md`
  - [x] **G4 step 4 (symbol audit) RAN on the Pi 2026-08-11** — needs no button presses. Two findings:
    - **The Pi's mixer speaks an older DPM API than this checkout, and Task 8 would have failed silently on it.** Pi: `get_dpm_states(start, end)` → `[[a, b, hold_a, hold_b, mono]]` and `enable_dpm(start, end, enable)`, living in `zyngine/zynthian_engine_audio_mixer.py`. WSL: `update_dpm_states()` + `mixer.dpm` and `enable_dpm(enable)`, in `zynlibs/zynmixer/zynmixer.py`. `updateDpmStates` does not exist on the Pi at all. The `hasattr` guard meant the meter would have degraded to fader position with no error. Fixed in `zynthian-ui` `f1c98493` — tries new, falls back to old. Both report dBFS
    - **RATCHET is unblocked** — `setStutterCount`/`setStutterDur`/`changeStutterCountAll` are all in the installed `libzynseq.so`, along with `setPlayChance`, `setNotePlayChance`, `setSwingAmount`/`setSwingDiv`. `addNote` is 5-arg (`_ZN7Pattern7addNoteEjhhff`)
  - [ ] **G4 steps 1, 2, 3, 5 still block deployment** — all four need someone pressing buttons on the panel. Steps 1/2/3 are the CC audit, step 5 the two SOLO gestures
  - [ ] **Pre-flight FAIL found 2026-08-11, fix before any CC audit:** `jack_lsp -c | grep -A3 "Pads MIDI"` shows **two** routes — `dev3_in` **and** `dev2_in`. Both appeared after a clean boot, so this is not the 2026-08-08 stale-`jack_connect` cause; suspect a device re-enumeration after a watchdog reopen taking a second zmip slot. Every CC arrives twice until it is resolved
  - [ ] `ZYNTHIAN_LOG_LEVEL` did not survive the reboot — re-set it to 20 before auditing, unset it after
  - [ ] **SP1 addendum — voice DENSITY (specced 2026-08-11, not built).** Spec: `docs/superpowers/specs/2026-08-11-sp1-addendum-voice-density.md`. The Turing voices write one note per step unconditionally; density adds rests. Mechanism: a **gate tap** — the register rotated by half its length, read as its own rotation sequence, with the N lowest-valued steps sounding where `N = round(density/100 * steps)`. Rank not threshold, so 100 is exactly every step and 0 is exactly none, and turning down only removes notes. Freezes with LOCK because the mask is a function of the register. **Reduces** the write burst. Two tasks, both pure WSL work, neither needs the Pi:
    - [x] **Task A1** — `0b5f1770`. `rotate`/`gate_values`/`gate_mask`, DENSITY spread page, `SPREAD_SPECS` entry, voice channel page column. **187 tests passing**, up from 164
    - [x] **Task A2** — `7d9c3584`. Mask in `_write_voice_pattern`, `density=100` default, snapshot persistence, encoder range, `GENERATOR_PARAMS`, dashed tab at density 0. `py_compile` OK, suite still 187
    - Verified with the real pure functions on a 16-step pattern: density 100/75/50/25/6/0 → 16/12/8/4/1/0 notes, and every mask is a subset of the one above it
    - **No `_verb` rejection branch was needed for drums** — `param_get` returns `None` for a key their state never had, and the encoder tail already bails on `None`
    - [ ] Hardware-verify on the panel: rides G4, nothing extra needed
    - Rejected and recorded, do not re-litigate: per-note `setNotePlayChance` (re-rolls every pass, breaks LOCK), euclidean mask (turns the voice into a drum channel with pitches), forcing step 0 to sound
  - 164 tests passing on WSL, **nothing pushed**, nothing on the Pi. SP1 code is complete; only hardware verification remains

- [ ] **Techno Machine pass two — SP2, SP3, SP4** — specced in §2 of the pass-two design, not built. SP2 live pad play + REC recording · SP3 the drum filter (**blocked on the Pi**) · SP4 channel type switching on SHIFT+GRID. Build order SP1 → SP2 → SP4, SP3 whenever the hardware returns

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
  - [x] Display: row-order puzzle **solved** 2026-08-09 — it was a wrong row stride, not dropped rows. A report is a 128x32 tile (16 bytes/row) and both row bands must be sent per tile. The "512x32 canvas / 2-px rows / discarded rows" model is dead
  - [x] Display: OSC drawing API built (`fbclear`/`text`/`rect`/`raw`) + Maschine-style layout photographed and readable — tabs, dotted rule, encoder columns with double-height values
  - [x] **Display geometry SOLVED and hardware-verified 2026-08-09** (`bbf2a62`) — 255x64 row-major, 8 reports of a full-width 8-row band. Header bytes 5 and 7 were **swapped**: byte 5 is bytes-per-row (0x20), byte 7 is rows (0x08). Byte 1 is an x offset in **bytes**, which is where "512 wide" came from. Taken from cabl's `MaschineMK2.cpp`, which had the answer all along. Full writeup: `MD/display-investigation.md`, first section
  - [x] Encoder indicator bars — built and verified in the mock: unipolar fill (HITS/LEN/EXPR/VOL), bipolar from centre (PAN), segments (ROT/DIV), y 52-61
  - [x] Layout wired into the ctrldev driver 2026-08-09 — group tabs with sample names, encoder columns, values and bars, all diffed so only changes go on the wire. Verified on hardware
  - [x] **Encoders are relative now, verified 2026-08-09** — per-group memory works. Three real defects found by measuring the CC stream, not guessing:
    - The daemon reported an absolute knob **position**, so one position served all eight groups: selecting another group and turning made its value jump to the previous group's. The value is now device state (`roller_value`) moved by deltas, with `/maschine/encoder` to re-centre it
    - The wrap guard `delta.abs() < 40` was too loose. Measured on the wire: real movement is **0-4 units per report**, counter wraps are **-38 to -40**, so wraps reached the host as real backwards movement. Threshold is 8 now
    - Tightening the guard alone made it **worse**: rejection skipped `set_roller_status`, so the baseline stayed stale by ~38, every later delta measured the wrap too and the encoder went dead. The baseline must resync on a rejected wrap
    - `zynthian_controller._set_value()` **truncates** integer controls (`:469`), so adding span/128 = 0.992 to pan never moved it - jumpy and uneven. Chain controls step in whole controller units with the remainder carried
    - Sensitivity: hits/rot = 128/(steps+1) - the sweep the absolute mapping used; div and length use a flat 8 units per step, because spreading their few settings over the sweep cost 26 and 32 units and read as sticky
  - [x] Pushed 2026-08-09 — `MaschineMK2_linux` main `b567fb0`, `zynthian-ui` vangelis `1ad9c8f0`, `zynth-docs` master
  - [x] **Per-group SFZ drum kits — DONE and hardware-verified 2026-08-09.** Eight LinuxSampler chains, kit on encoder 7, sample on encoder 6, volume/pan moved to the mixer strip. Snapshot `021-maschine-drum-rig-sfz` (built programmatically from `020`, patterns byte-identical). Measured: 6.2% CPU, 249.5 MB, zero xruns, kits survive a restart. Spec + plan in `docs/superpowers/`; the SDD ledger has every finding
    - **Cost spike done 2026-08-09 on the Pi, resources are a non-issue.** Standalone `linuxsampler` over LSCP, 8 different kits (TR808/909, LINN9000, SP1200, Simmons, CR78, HR16, RX11): **248 MB RSS total**, ~1.5 MB per extra kit, **0.06 s** mean load, **~0 s** live kit swap on a running channel, **14% of one core** with all 8 triggered at 16th notes. 3.0 GB still free. Script: `/root/sfz_spike.py`, touches no Zynthian state
    - Zynthian's LS engine is **one shared process** (`jackname = "LinuxSampler"`, one 32-channel JACK device, processors are `LinuxSampler:outN_`), so that 248 MB covers all eight groups, not per group
    - Remaining work is mapping, not resources: (a) sample NAMES — tabs resolve via `keymaps.json` on the preset path, which no SFZ kit matches, so parse the `.sfz` (`key=`/`lokey=` + `sample=`) and build each kit's own note list; (b) a group's current note may not exist in the new kit, so pick a sensible landing note; (c) encoder **7 is free and its screen column is already blank** — natural home for kit select, with the load debounced off the MIDI thread
    - Kits persist for free: engine + preset live in the `.zss`
  - [ ] Unset `ZYNTHIAN_LOG_LEVEL` on the Pi once task 10 is done: `systemctl unset-environment ZYNTHIAN_LOG_LEVEL`
  - [ ] **jackd is on the wrong soundcard (found 2026-08-09).** `jackd` runs `-d alsa -d hw:Headphones -r 48000 ...` and `zynthian_envars.sh` has `SOUNDCARD_NAME="RBPi Headphones"` — the Pi's built-in PWM output, not the documented Sound Blaster Play! 2 (`hw:S2`, 44.1 kHz). The SFZ kit rig's 6.2% CPU / zero-xrun measurement is real but taken on the wrong card, so it is not representative. Fix the soundcard in webconf, then re-measure before trusting it for FX headroom (this blocks techno-machine Gate G1 below)
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

- [~] **Techno Machine — 5 euclidean drums + 3 Turing voices, played from the Maschine (built 2026-08-10)**
  - Sub-project of the Maschine rig. **All three gates passed and the prototype is implemented; only the twenty-minute jam remains.**
  - [x] **Gate G1 — FX cost.** Re-baselined: the spec's "10% of one core for sixteen inserts" is unreachable, because sixteen jalv processes cost **16.5% of a core doing nothing**. Measured in the real topology with the rig sounding: **20.7% JACK DSP load, zero xruns**, MemAvailable −177 MB, 3.8 s warm start for the sixteen. Owner waived the `hw:S2` precondition — every number is on `hw:Headphones` at 48 kHz and must be re-measured when the Sound Blaster is connected
  - [x] **Gate G2 — engines.** JC303, Obxd and padthv1 all expose all four CONTROL columns; symbol table recorded and wired into `techno_lib.VOICE_SYMBOLS`
  - [x] **Gate G3 — wet parameter.** **MDA Ambience and MDA DubDelay, the spec's own starting choice, are both dry/wet crossfades**, as are PlateX2, MDA Delay, lcrDelay and bolliedelay. Chosen instead: **TAP Reverberator + TAP Stereo Echo**, true wet levels, stereo in and out, verified on hardware — the dry survives a full sweep
  - [x] Prepared snapshot **`016-techno_maschine`**: 5 LinuxSampler drums + JC303/Obxd/padthv1, sixteen post-fader inserts, dry at unity and wet at −70 dB, strips 0.19 / main 0.80 for headroom
  - [x] Driver: state dict with one `apply()` path, per-channel FX handles, three latched pages, drum STEP page, the Turing voices, voice CONTROL page, ALL page with ganged FX, mute/solo/erase, snapshot persistence of the registers
  - [ ] **SOLO needs a closer look — deferred by the owner 2026-08-11.** Mute (tap and hold), ERASE and Restart all verified on hardware; the two SOLO gestures were not. Check both: SOLO held + Fn as a momentary solo, and SOLO tapped as a latched mode where the F row becomes solos. Worth knowing before testing: `zynmixer.toggle_solo` is **additive**, not exclusive, and it has a special case at `MAX_NUM_CHANNELS - 1` where the main strip clears every solo. Exclusive solo (SHIFT + Fn) was always pass two, because SHIFT does not emit yet
  - [x] **Task 14 — the twenty-minute jam PASSED 2026-08-11.** DSP load mean 21.1% / p95 37% / max 45%, **zero xruns, zero segfaults, zero tracebacks**, MemAvailable flat (2639 → 2631 MB), watchdog reopens one per 22.6 s against a healthy ~8 s baseline. Three Turing voices rewriting a pattern every ~0.6 s for twenty minutes retires risks R1 and R6
  - [x] Two defects the jam itself exposed, both fixed: a preset load ran on the MIDI thread under the lock and **froze the whole instrument** (deferred to the poll thread, as kit loads already were); and a voice silenced by play chance 0 had no surface indication and read as a hang (its tab now draws dashed)
  - [ ] Re-measure G1 on `hw:S2` at 44.1 kHz once the external soundcard is connected
  - [ ] Tutorial page for the techno machine (and the older drum-rig tutorial debt still stands)
  - Plan: `docs/superpowers/plans/2026-08-10-techno-machine-prototype.md` · Gates: `docs/superpowers/techno-machine/2026-08-10-gates-g1-g2-g3-results.md`
  - **Retracted finding, do not re-derive:** JC303 and Obxd are **omni**. The claim that they answer only on MIDI channel 1 came from a probe that was not reset between channel rounds, plus the fact that an unconfigured `ZynMidiRouter:devN_in` routes to the **active chain** rather than per channel. No channel translation is needed

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
