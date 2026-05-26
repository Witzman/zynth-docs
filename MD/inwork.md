# Zynthian — In Work

Read this after CLAUDE.md.

Status: `[~]` drafting · `[t]` user testing · `[>]` ready to publish · `[ ]` future candidate

---

## Tutorials

- [~] **Custom MIDI Channel Routing** — Xboard channels 1–4 each drive a dedicated chain; snapshot persists on boot
- [~] **SMC-PAD Launcher Control** — 16 pads trigger 4×4 sequencer grid via MIDI master channel + CUIA TOGGLE_SEQ
- [~] **ESI U46DJ USB Audio Setup** — connect U46DJ to Zynthian, configure at 44.1 kHz (4in/6out), verify output + inputs
- [~] **Maschine MK2 Controller** — connect and map NI Maschine MK2 as MIDI controller in Zynthian
- [~] **Generative Drone Synth** — self-evolving ZynAddSubFX drone, SMC-PAD pads shift root note, 8 knobs shape texture
- [~] **Audio FX Chain with MOD-UI** — route ESI mic (CH 1/2) and line (CH 3/4) inputs through MOD-UI pedalboard, output on ESI
- [~] **Multi-Controller Performance Rig** — Xboard + SMC-PAD + Maschine MK2 simultaneous roles, saved as performance snapshot
- [~] **EMU Xboard CC Knob Mapping** — static and follow-channel CC bindings across 4 chains; depends on MIDI Channel Routing tutorial


## Documentation Updates

<!-- Add active items as: - [~] **Title** — one-line description -->

---

## Future Tutorial Candidates

- [ ] **Live Looper with SooperLooper** — record drone/instrument layers live, SMC-PAD transport controls record/play/overdub
- [ ] **ZynAddSubFX Sound Design from Scratch** — build custom evolving pad from ADD + PAD synth modules, no presets
