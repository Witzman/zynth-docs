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

- [ ] **Test Maschine MK2 Step Sequencer Part 1 on Pi**
  - [ ] Confirm sequencer mode activates (Shift+Pad Mode twice)
  - [ ] Confirm pads toggle steps (no note fired)
  - [ ] Confirm Play starts sequence, Erase stops it
  - [ ] Confirm MIDI Ch2 reaches Zynthian chain
  - Tutorial file: `~/zynth-docs/htmldoku/project-maschine-step-sequencer.md`

---

- [ ] **Test Dub Techno Live Rig — Maschine Pad Layer Part 1 on Pi**
  - [ ] Blocked: Maschine Step Sequencer Part 1 must pass first
  - [ ] Add pad chain on Ch2, verify 3-layer playback simultaneous
  - [ ] Test tempo drift over 8+ bars — document acceptable window
  - [ ] Test Shift+encoder B6 speed control (marked "under construction")
  - Tutorial file: `~/zynth-docs/htmldoku/project-dub-techno-maschine-pad.md`

---

## Backlog

<!-- Add future cross-cutting tasks here -->
