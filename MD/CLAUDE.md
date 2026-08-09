# Zynthian — Claude Context

## Read First

**On every session start, read all four files in this order before doing anything else:**

1. This file (`CLAUDE.md`) — project layout, hardware, file locations
2. `MD/inwork.md` — active tutorials and future candidates
3. `MD/todo.md` — active tasks and tutorial completion work
4. `MD/agent-behavior.md` — all session rules, workflow, and writing rules (read in full)

`MD/agent-behavior.md` governs all agent behavior. Do not proceed until all four are read.

---

## RESUME HERE — Techno Machine prototype (paused 2026-08-10)

**The live thread is the techno-machine prototype spec, not the drum rig.** The
drum rig itself (tasks 1-9, plus per-group SFZ kits) is done and hardware-verified.
Its only remaining debt — the oldest outstanding item in this project — is the
**tutorial page**, deferred since 2026-08-08; pick that up if the techno-machine
gates are blocked or as a break from them, but the techno-machine work is what
this session should drive.

**Read before doing anything on the techno machine:**

| What | Where |
|---|---|
| Prototype spec — the live document | `docs/superpowers/specs/2026-08-10-techno-machine-prototype-design.md` |
| Design synthesis | `docs/superpowers/techno-machine/2026-08-09-techno-machine-design.md` |
| PO / dev debate positions | `docs/superpowers/techno-machine/po-position.md`, `dev-position.md` |
| Drum rig progress ledger (context, not the active task) | `~/zynth/zynthian-ui/.superpowers/sdd/2026-08-06-maschine-drum-rig/progress.md` |

**State (2026-08-10):** the prototype is designed and six contested decisions are
ratified by the owner (Turing lock incremental-not-rewrite with 4-deep undo; sends
are ganged per-channel post-fader inserts, cheapest plugins first; big encoder
dropped for verb-button + the eight small encoders; F1-F8 = MUTE / SOLO button,
**no daemon work needed**; Lock snapshots deferred to pass two; `setPlayChance`/
`setSwingAmount` ship, `setStutterCount` is pass two, `setNotePlayChance`/
`addControl` refused). **No implementation plan exists yet** — the spec defines
three gates that must run first:

1. **Gate G1 — FX cost** (16 new plugin processes: RSS, JACK graph nodes, snapshot
   load time). **Blocked on the jackd/soundcard bug below** — measuring on the
   wrong card makes G1 meaningless.
2. **Gate G2 — voice engines' controller lists.** LinuxSampler already taught this
   project that "enabled" says nothing about what a chain exposes; verify before
   designing the voice CONTROL page.
3. **Gate G3 — wet parameter.** Each FX plugin's wet control must be a true wet
   level, not a dry/wet crossfade (that would break the encoders-7/8-are-sends
   contract).

**First action next session: fix the jackd/soundcard mismatch, then run Gate G1.**
`jackd` on the Pi runs `-d alsa -d hw:Headphones -r 48000 ...` (the Pi's built-in
PWM output) and `zynthian_envars.sh` has `SOUNDCARD_NAME="RBPi Headphones"` — not
the Sound Blaster Play! 2 (`hw:S2`, 44.1 kHz) this project's hardware notes
describe. The 2026-08-09 SFZ-kit measurement (6.2% CPU, zero xruns) is real but
was taken on the wrong card, so it cannot be trusted for FX headroom. Fix the
soundcard in webconf, re-measure, then run G1.

Deployed HEADs:

| Repo | Branch | HEAD | Pushed? |
|---|---|---|---|
| `MaschineMK2_linux` | main | `b567fb0` | yes |
| `zynthian-ui` | vangelis | `75572577` | yes |
| `zynth-docs` | master | `dc6cd5c`+ | yes |

**Per-group SFZ drum kits shipped (2026-08-09).** All eight groups run
LinuxSampler from snapshot `021-maschine-drum-rig-sfz`; each picks its own
drum machine on encoder 7 and its sound within that kit on encoder 6. Kit
notes and names are parsed from the `.sfz` files - Zynthian's `keymaps.json`
resolves on the synth's preset path and cannot match an SFZ kit. **Volume and
pan moved to the MIXER STRIP** because `zynthian_engine_linuxsampler` defines
no controllers at all (`_ctrls = []`), so reading them off the engine dies the
moment a group runs a kit; expression is gone with no equivalent. Measured on
the rig: 6.2% system CPU, 249.5 MB sampler RSS for all eight kits, zero xruns,
kit switching mid-jam with no glitch, and kits survive a restart. `020` is kept
as the FluidSynth fallback. Spec and plan: `docs/superpowers/{specs,plans}/2026-08-09-maschine-sfz-kits*`.

**Encoders are relative (2026-08-09).** The daemon holds each encoder's CC value as device state (`roller_value`) and moves it by the hardware delta; `/maschine/encoder idx value` re-centres it. A knob's *position* cannot serve eight groups - mapping it straight onto a parameter made every group share one value. Measured facts, do not re-derive: real movement is **0-4 units per report**, counter wraps are **-38 to -40**, so the wrap guard is 8; a rejected wrap must still resync `roller_status` or the encoder goes dead; and `zynthian_controller._set_value()` **truncates** integer controls, so chain controls must step in whole controller units with the remainder carried, never in fractions of the range.

**Control layout as shipped** (differs from the plan — see the ledger for why): pads toggle steps · Group A-H select · enc 1 hits, 2 rotation, 3 division, 4 length, 5 pan (**mixer balance**), 6 **sample within the kit**, 7 **kit**, 8 volume (**mixer level**) · F1-F8 mute groups A-H regardless of selection (mixer strip mute) · Play toggles all 8 · Restart to step 0 · Erase clears the selected group · **the arrows beside the display** change sample. Group buttons carry their group's colour with brightness showing its volume.

**Lessons from the 2026-08-08 test round — do not relearn:**

- **The daemon swallows its Page ◀▶ buttons** (CC 48/47) for its own page indicators and never emits them. The arrow buttons beside the display are `step_left`/`step_right`, **CC 5/6**. Dump `a2j:...Pads MIDI` with `jack_midi_dump` before binding any button.
- **LOOP play mode must be re-forced, not set once.** Restoring a snapshot rewrites every sequence's play mode from the `.zss`, and the prepared snapshot carries LOOPALL. A LOOPALL sequence shorter than the bar goes RESTARTING at its own end, which the next non-sync clock turns into STARTING, and STARTING does not clock its tracks — the group falls silent until the next bar sync instead of looping.
- **zynseq cannot persist a mute.** Its track record stores type, chain id, channel, output, map and the pattern list, nothing else. Mute the **mixer strip** instead; that is in the snapshot and shows on the touchscreen mixer.
- **Nothing repaints the LEDs after a snapshot load** unless the driver registers `SS_LOAD_SNAPSHOT`, re-derives its cached params and **clears the LED cache** — otherwise the repaint is suppressed as unchanged.
- **Pattern length is quantised to whole beats and always will be**: `getLength() = beats * PPQN`, and there is no `setSequenceLength` in the installed C API. Reachable step counts are `beats * steps_per_beat`; 1, 5, 7, 11, 13 are unreachable with the current five divisions.
- **Phantom drum sounds on pad taps = a stale JACK route.** `zynautoconnect` only tears down connections it made itself, and jackd outlives a zynthian restart, so a manual `jack_connect` from a debugging session lives forever. Check `jack_lsp -c | grep -A3 "Pads MIDI"` — it must show exactly **one** `devN_in`.
- **In webconf's Snapshots page, the Name field + checkmark RENAMES THE SELECTED BANK.** It does not save a snapshot; it renamed bank `000`. Save from the touchscreen: inside a bank, the first entry is **"Save as new snapshot"**.

**Display geometry is SOLVED and hardware-verified (2026-08-09, `bbf2a62`).** 255x64 per screen, 1bpp row-major, MSB leftmost, 32 bytes per row. One screen = 8 reports, each a full-width band of 8 rows: header `[0xE0|s, 0, 0, chunk*8, 0, 0x20, 0, 0x08, 0]` + 256 payload bytes sliced straight out of the framebuffer. Header bytes 5 and 7 were **swapped** in this driver - byte 5 is bytes-per-row, byte 7 is rows - which is why every screen garbled. Byte 1 is an x offset in **bytes**, which is where the wrong "512 wide" came from. Source of truth: cabl `src/devices/ni/MaschineMK2.cpp`. The layout is wired into the ctrldev driver: group tabs with sample names, dotted rule, encoder columns with double-height values and indicator bars.

Full writeup: `MD/display-investigation.md`, first section — read it before touching the display. The daemon exposes an OSC drawing API (`/maschine/display/fbclear|text|rect|raw`) so the layout lives in the driver, not in Rust. Screen 0 = left `0xE0`, 1 = right `0xE1`; flushed on the 100 ms timer, never from the OSC handler — drawing from the handler puts HID writes on the fd input arrives on and trips the hidraw watchdog. Left = A-D + HITS/ROT/DIV/LEN, right = E-H + PAN/EXPR/VOL. **Do not fill a box and then draw inverted text into it** — the two cancel; draw the text, then invert the whole tab.

**Hard-won facts — do not relearn these:**

- The Pi's installed Zynthian and `libzynseq.so` are **older** than the `~/zynth/zynthian-ui` checkout. Never write a `libseq.*` call from local headers; audit it first with
  `ssh root@192.168.2.123 'nm -D --defined-only /zynthian/zynthian-ui/zynlibs/zynseq/build/libzynseq.so | awk "\$2==\"T\"{print \$3}" | sort'`
  This has already broken three times (call arity, `clearPattern`, `getNoteAtIndex`).
- zynseq addresses sequences as `(bank, sequence, track)` via `self.zynseq.bank`; there is no scene/phrase level. `getSteps()`, `getClocksPerStep()`, `setStepsPerBeat()`, `setBeatsInPattern()` and `clear()` take no pattern argument — `selectPattern()` first. Pattern ids in the prepared snapshot are 10-17, so always resolve via `getPattern(...)`.
- The Pi **cannot fetch from GitHub** (root has no auth). Move commits with `git bundle create` on WSL, then `git fetch /tmp/x.bundle main:refs/remotes/origin/main` on the Pi. A bare `git reset --hard origin/main` there once rewound the tree because the fetch had silently failed — check fetch exit status first.
- `~/zynth-docs/tools/patch-autoconnect-maschine.py` must be **re-run after any Zynthian update**, or Zynthian never gives the daemon's virtual port a zmip slot and the driver is "Found" but never "Loaded" — the rig then does nothing at all, with no error.
- MK2 input dying after seconds was a kernel hidraw fault, fixed by a close-then-reopen watchdog in the daemon (`MaschineMK2_linux` `0b36cd9`). Full diagnosis and everything already ruled out: `htmldoku/project-midi-reference.md`, "Conflict 12". Journal lines `watchdog: input stalled, reopened ...` every ~8s are healthy.
- Step 0 is the **top-left** pad. LED index for a step is `PAD_OFFSETS[step]` with `PAD_OFFSETS = [12,13,14,15,8,9,10,11,4,5,6,7,0,1,2,3]` (its own inverse). The daemon's `set_rgb_light` halves brightness, so the driver passes 2.0 for full.
- **Every zynseq call the driver makes must hold `self.lock`.** `libzynseq` is not thread-safe and the driver reaches it from three threads (MIDI handler, zynsigman queued handler, 30 Hz playhead poll). Without the lock the whole Zynthian UI died with SIGSEGV, exit 139, about 95s into a jam.
- **Never drive anything step-rate-sensitive from `SS_SEQ_PROGRESS`** — it is 5 Hz (`slow_thread_task`, 0.2s sleep) and aliases against the step rate, which skipped playhead pads unpredictably. Poll in your own thread and cache clocks-per-step so the hot path never calls `selectPattern()` (that writes zynseq's single global pattern selection and fights the pattern editor for it).
- **`TOGGLE_PLAY` is not a sequencer transport** — it resolves to `cuia_toggle_audio_play()`, which toggles the audio file player, or just the one pattern if the pattern editor is on screen. Use `setPlayState` on every sequence; it starts JACK transport itself.
- **`"external_pad_leds": true` must stay in the daemon's `maschine.json`** or the daemon repaints pads on press/release in its own global colour and the first touch destroys the per-group picture. It is not in git on the Pi — `git reset --hard` there wipes it, so re-set it after every deploy.
- `chain_manager.get_chain_ids_by_midi_chan()` **does not exist** on the Pi. Use `chain_manager.midi_chan_2_chain_ids[chan]`, as its own pattern editor does.
- **Filter control on FluidSynth drum kits is a dead end** — CC 74/71 are unipolar SoundFont modulators that only *add* to `initialFilterFc`, and `FluidDrums.sf2` ships wide open at 13500 cents. There is no pitch/tune controller either. Real filtering needs an LV2 filter per chain.
- `encoder_step` in the daemon receives the encoder's **absolute counter byte**, not a delta (`mikro.rs:415` passes `byte as i32`); `send_encoder_cc` divides it by 4. Treating it as a delta pins anything driven by it instantly.
- Redrawing the display per input report starves the input reader and trips the hidraw watchdog. Rate-limit to the existing 100ms display timer.
- LED report byte layout is now measured, not guessed — see `MD/display-investigation.md` and the ledger. `/maschine/rawled` and `/maschine/display/test|opts|calib|clear` exist for mapping more of it.
- `ZYNTHIAN_LOG_LEVEL=20` is currently set on the Pi so ctrldev load lines are visible. Clear it with `systemctl unset-environment ZYNTHIAN_LOG_LEVEL` once debugging is done.

**Open items not yet in the plan:** the display's vertical row mapping (above); per-group *kit* switching across the 42 drum-machine SFZ kits in `/zynthian/zynthian-data/soundfonts/sfz/Drum Machines/`, which would beat any CC for character; `light_buf2` bytes 17-31 and several `light_buf3` transport bytes still unverified; and sub-project 2 (two Turing-machine voices on the SMC-PAD) has no spec yet.

**Closed 2026-08-08:** group-button RGB layout is mapped (full RGB triplets, starts 1, 7, 13, 22, 25, 34, 37, 46) and the cold-boot ordering race survived a real power cycle with the alias present and bound. One sample only — if it recurs, the fix is `After=maschine-mk2.service`.

---

## Working Rules

General engineering behavior. `MD/agent-behavior.md` remains authoritative for tutorial process, tone, and writing rules.

1. Resolve ambiguity from code, docs, and git history first; ask only when the answer changes what you build and you can't derive it. Unattended: pick the most reasonable interpretation, proceed, log the assumption.
2. Match solution weight to problem weight — no abstraction or flexibility for a need that doesn't exist yet (YAGNI). Justify any indirection in one sentence or drop it.
3. Don't touch unrelated code. Surface design smells and out-of-scope bugs separately (note, don't fix inline) so we triage them as their own issue.
4. Flag uncertainty explicitly — confidence without certainty does more damage than admitting a gap. When a question is testable, run a small low-risk experiment and bring hypothesis + result instead of guessing.
5. Propose structurally better paths — especially lasting over tactical — before implementing the obvious patch. Brief: what, why better, the cost.
6. Verify before claiming done. Run it, show the output. Evidence before assertions.

---

## Role

Tutorial generator and teacher. Session rules, workflow, and writing rules in `MD/agent-behavior.md`.

---

## Hardware Setup

| What | Detail |
|------|--------|
| Hardware | Raspberry Pi 4, ZynthianOS |
| Kit / encoders | None — no physical hardware kit |
| Touchscreen | Elecrow ESP32 5" 800×480 — HDMI (video) + USB (touch). Working. |
| Access methods | Touchscreen (V5 keypad active) · SSH · webconf (`http://zynthian.local`) · VNC |
| SSH | `ssh root@192.168.2.123` — SSH key configured in WSL (mDNS `.local` does not resolve from WSL2) |
| Audio interface | Creative Sound Blaster Play! 2 — USB, 2-in 2-out, 44.1 kHz, ALSA name `hw:S2`. Temporary replacement for U46DJ (buggy USB hub issues). |
| MIDI keyboard | E-MU Xboard (25/49/61 series) — USB/MIDI keyboard controller, 16 CC knobs, pitch/mod wheels, aftertouch |
| MIDI pad controller | SMC-PAD — 16-pad USB/Bluetooth controller, 8 encoders, DAW transport, note repeat |

Tutorial navigation uses V5 touch keypad (primary), SSH, and webconf. Never assume physical encoders or hardware kit buttons. Touchscreen (V5 keypad at left) is active and working — use it as the primary navigation method. Webconf stays only for: MIDI port enable/disable, audio/JACK settings.

---

## Physical Layout

```
~/zynth/                       ← project root (NOT a git repo)
    zynthian-sys/              ← git repo: system scripts, config, boot, sbin
    zynthian-ui/               ← git repo: Python UI + synth engine
    zynthian-webconf/          ← git repo: web config interface (Python/Tornado)
    zynthian-hw/               ← git repo: hardware — PCBs, schematics, parts specs, pin assignments
    MaschineMK2_linux/         ← git repo: Rust HID daemon for Maschine MK2
    CE/                        ← NI Controller Editor 2.7.6 (Windows install) — MK2 factory MIDI-mode templates
    SMC Pad/                   ← NiFox preset pack — SMC-PAD as Koala Sampler controller (iPad), not Zynthian
    manuals/                   ← controller and interface manuals (txt/pdf)
    MD/   →  symlink           ← ~/zynth-docs/MD/
    CLAUDE.md  →  symlink      ← ~/zynth-docs/MD/CLAUDE.md

~/zynth-docs/                  ← git repo: documentation
    MD/                        ← session tracking files (NOT documentation)
    htmldoku/                  ← documentation SOURCE (.md files, hand-edited)
    docs/zynthian-Doku/        ← rendered HTML — generated by generate-html.py
```

Tutorials go in `htmldoku/` under the **Personal Projects** section. The sidebar entries are configured in `generate-html.py`.

**Publishing a tutorial means: edit `htmldoku/*.md`, run `generate-html.py`, commit both, push.**

```bash
cd ~/zynth-docs
python3 htmldoku/generate-html.py
git add htmldoku/mytutorial.md docs/zynthian-Doku/mytutorial.html
git commit -m "docs: add tutorial — <title>"
git push
```

---

## Reference Index

Read the relevant page **before drafting any tutorial step** on that topic.

| Topic | Read |
|-------|------|
| First boot, SD flash, initial access | `~/zynth-docs/htmldoku/getting-started.md` |
| Understanding Zynthian concepts | `~/zynth-docs/htmldoku/userguide.md` |
| Synth engines, loading presets, engine params | `~/zynth-docs/htmldoku/synth-engines.md` |
| MIDI controllers, routing | `~/zynth-docs/htmldoku/midi.md` |
| E-MU Xboard controller details | `~/zynth/manuals/EMU Xboard 25_49_61 manual Eng.txt` |
| SMC-PAD controller details | `~/zynth/manuals/SMC-PAD_ZE.txt` |
| U46DJ audio interface details | `~/zynth/manuals/U46DJ-English.pdf` |
| Audio device, JACK setup | `~/zynth-docs/htmldoku/audio.md` |
| Snapshots, saving/restoring state | `~/zynth-docs/htmldoku/snapshots.md` |
| Webconf interface reference | `~/zynth-docs/htmldoku/webconf.md` |
| Display, wiring, hardware variants | `~/zynth-docs/htmldoku/hardware.md` |
| Common user setups (layering, looper, etc.) | `~/zynth-docs/htmldoku/recipes.md` |
| Troubleshooting errors | `~/zynth-docs/htmldoku/troubleshooting.md` |
| Quick Q&A / FAQ | `~/zynth-docs/htmldoku/faq.md` |
| System architecture, boot sequence | `~/zynth-docs/htmldoku/architecture.md` |
| All env vars / configuration variables | `~/zynth-docs/htmldoku/configuration-reference.md` |
| LV2 plugin install and management | `~/zynth-docs/htmldoku/lv2-plugins.md` |
| PCBs, schematics, parts specs, pin assignments | `~/zynth/zynthian-hw/` — per-board dirs (`V5_main`, `ZynScreen_v1.5`, `Zynaptik_v3`, …), plus `doc/`, `lib/`, `fritzing_parts/` |
| Maschine MK2 factory MIDI-mode map, pad LED HSB model | `~/zynth-docs/htmldoku/project-midi-reference.md` §"factory MIDI mode" — raw source is `~/zynth/CE/*.ncc` (plain XML) |
| SMC-PAD Koala Sampler preset pack | `~/zynth-docs/htmldoku/project-midi-reference.md` §"NiFox preset pack" — raw source is `~/zynth/SMC Pad/` |

---

## Tracking Files

| File | Purpose |
|------|---------|
| `MD/agent-behavior.md` | Agent rules — tutorial creation process, tone, format, writing rules |
| `MD/inwork.md` | Active tutorials — `[~]` drafting · `[t]` testing · `[>]` ready to publish |
| `MD/todo.md` | Active tasks and tutorial completion work — read every session |
| `MD/done.md` | Published tutorials `[x]` |
| `MD/bugs.md` | Issues found during tutorial testing |

