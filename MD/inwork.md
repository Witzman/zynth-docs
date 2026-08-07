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

- [~] **Maschine MK2 Drum Rig** — 8 groups x 16 steps euclidean drum sequencer via a new ctrldev driver; zynseq holds the patterns. NOT a tutorial yet — implementation plan in progress, see the RESUME HERE section of CLAUDE.md. Tasks 1-9 done and hardware-verified (2026-08-08); **resume at task 10** (snapshot round-trip, tutorial page, tracking files). First action: user hardware-tests `2fc6a837` — pattern length/polyrhythm, sample switching, encoders 5-7.

---

## Future Tutorial Candidates

- [ ] **ZynAddSubFX Sound Design from Scratch** — build custom evolving pad from ADD + PAD synth modules, no presets
- [ ] **MaschineMK2_linux MIDI Clock Sync** — add ALSA MIDI clock input to daemon so Zynthian transport drives Maschine step rate; eliminates drift; requires Rust code contribution to MaschineMK2_linux
