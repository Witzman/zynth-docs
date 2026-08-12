# Zynthian — In Work

Read this after CLAUDE.md.

Status: `[~]` drafting · `[t]` user testing · `[>]` ready to publish · `[ ]` future candidate

---

## Tutorials

- [>] **MIDI Reference** — published; update after each tutorial verification
- [~] **Custom MIDI Channel Routing** — Xboard channels 1–4 each drive a dedicated chain; snapshot persists on boot
- [~] **SMC-PAD Launcher Control** — 16 pads trigger 4×4 sequencer grid via MIDI master channel + CUIA TOGGLE_SEQ
- [~] **SMC-PAD Drum Computer** — 16 pads as live GM drum kit + step-sequenced beat launcher; pads 13–16 launch patterns
- [~] **ESI U46DJ USB Audio Setup** — connect U46DJ to Zynthian, configure at 44.1 kHz (4in/6out), verify output + inputs
- [~] **Maschine MK2 Controller** — Parts 1+3 verified; Part 2 (CC Learn, now unblocked — encoders send standard CC); Part 4 (web editor, MIDI IN, display) draft
- [~] **Generative Drone Synth** — self-evolving ZynAddSubFX drone, SMC-PAD pads shift root note, 8 knobs shape texture
- [~] **Audio FX Chain with MOD-UI** — route ESI mic (CH 1/2) and line (CH 3/4) inputs through MOD-UI pedalboard, output on ESI
- [~] **Multi-Controller Rig (rig-v1)** — SMC-PAD drums (ch6) + transport ctrldev, Maschine seq→bass (ch2), Xboard strings/lead (ch3/ch4); replaces old performance-rig tutorial
- [~] **EMU Xboard CC Knob Mapping** — static and follow-channel CC bindings across 4 chains; depends on MIDI Channel Routing tutorial
- [~] **Dub Techno Performance Loop** — drums + bass + pad via step sequencer, delay/reverb effects, SMC-PAD live mute control
- [~] **Maschine MK2 Step Sequencer** — Parts 1+5 verified; Part 2 (pages + per-step note/vel) blocking Part 4 (euclidean); Part 3 (tempo encoder) uncertain — daemon marks it "under construction"
- [~] **Dub Techno Live Rig — Maschine Pad Layer** — Maschine Ch2 pad layer over Zynthian drum+bass; dub delay+reverb; live step toggle techniques; prereqs: Dub Techno Part 1 + Maschine Step Sequencer Part 1
- [~] **Live Looper with SooperLooper** — synth + U46DJ live audio into SooperLooper; SMC-PAD transport controls record/play/overdub; pads trigger individual loop slots

---

- [~] **Maschine MK2 Drum Rig** — 8 groups x 16 steps euclidean drum sequencer via a new ctrldev driver; zynseq holds the patterns. NOT a tutorial yet. Tasks 1-9 done and hardware-verified; per-group SFZ drum kits shipped and hardware-verified 2026-08-09 (this part is now complete). **COMPLETE. The tutorial page was dropped by the owner 2026-08-12** — the drum rig is superseded by the techno machine, which has its own manual and will get the tutorial page instead. Everything in task 10 is done and pushed. See RESUME HERE in CLAUDE.md.

- [x] **Techno Machine — moved OUT of this repo 2026-08-13.** The tutorial page `project-techno-machine.md` was deleted along with its sidebar entry; the build guide now lives in its own self-contained repository, **[Generative-Techno-ZynthianMaschine-MKII](https://github.com/Witzman/Generative-Techno-ZynthianMaschine-MKII)**, published at <https://witzman.github.io/Generative-Techno-ZynthianMaschine-MKII/>. That repo carries the guide, the vendored Rust daemon, the ctrldev driver and its 271 tests, the system files, the tools and the factory snapshot `017-generative-techno`, so a reader needs one checkout. Design and plan: `docs/superpowers/specs/2026-08-13-generative-techno-repo-design.md` and `docs/superpowers/plans/2026-08-13-generative-techno-repo.md`

- [~] **Techno Machine** — sub-project of the drum rig: 5 euclidean drum channels + 3 Turing-machine synth voices (bass/lead/pads), per-channel reverb and delay, played entirely from the Maschine. NOT a tutorial yet. Designed 2026-08-09/10, **all three gates passed and the prototype implemented and deployed 2026-08-10**. Prepared snapshot `016-techno_maschine`; driver in `zyngine/ctrldev/`. **The twenty-minute jam passed 2026-08-11 — zero xruns, zero segfaults, zero tracebacks.** SOLO was closed 2026-08-11 by observation. Remaining: a re-measure on `hw:S2` once the Sound Blaster is connected, and the tutorial page. Manual at `~/zynth/TECHNO-MACHINE-MANUAL.md`. Spec, gates, plan and manual under `docs/superpowers/`

- [~] **Techno Machine pass two** — the owner's ten-feature list, decomposed into SP1-SP5. NOT a tutorial yet. **SP1, SP5 and SP2 are all built, pushed, deployed and hardware-verified; 248 tests.** SP3 is next and unblocked, SP4 last.
  - **SP1 — mode & page framework** (five latched modes, DL/DR page rings, spread pages): shipped and verified 2026-08-11, alongside **SP5 — pattern time** (the `1/4` division, notes up to eight steps). 23 hardware checks, five defects found and four fixed the same day.
  - **SP2 — live pad play and REC recording**: shipped and verified **2026-08-12**, 8 hardware checks, zero defects. Pads are the instrument in every mode but STEP; REC held overdubs with hold-time note length; a durable `owner` flag with two handback routes; played-in steps light amber.
  - **SP3 — drum filter**: gate passed 2026-08-11, **MDA RezFilter** chosen. Needs its own spec and plan. Trap: below `freq` 35 it emits exact digital silence, so the encoder must map onto 35-100. Open: per-chain insert versus a shared drum bus.
  - **SP4 — channel type switching** on SHIFT+GRID: last, because its ownership rules are defined by SP2's `owner` flag.
  - Specs and plans dated 2026-08-11 and 2026-08-12 under `docs/superpowers/`; gate results and test findings under `docs/superpowers/techno-machine/`

---

## Future Tutorial Candidates

- [ ] **ZynAddSubFX Sound Design from Scratch** — build custom evolving pad from ADD + PAD synth modules, no presets
- [ ] **MaschineMK2_linux MIDI Clock Sync** — add ALSA MIDI clock input to daemon so Zynthian transport drives Maschine step rate; eliminates drift; requires Rust code contribution to MaschineMK2_linux
