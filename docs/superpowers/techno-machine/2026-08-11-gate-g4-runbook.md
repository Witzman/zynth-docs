# Gate G4 — Maschine MK2 Surface Audit Runbook

**Why this exists:** every CC in the SP1 plan
(`docs/superpowers/plans/2026-08-11-techno-machine-pass-two-sp1.md`) is read out
of the daemon's source, not observed on the wire. G4 converts them from claims
into measurements. **G4 blocks deployment of pass two**, not local development.

**Access:** SSH to the Pi (`ssh root@192.168.2.123`) plus physical access to the
Maschine MK2 — steps 1, 2, 3 and 5 need someone pressing buttons.

**Status:** steps 0 and 4 ran 2026-08-11 and are recorded below. Steps 1, 2, 3
and 5 are pending — they cannot run without hands on the panel.

---

## Step 0 — Pre-flight, before any measurement

Run all four. Each has already cost this project a session.

```bash
# a. Exactly ONE devN_in on the pads port, or every pad tap doubles
ssh root@192.168.2.123 'jack_lsp -c | grep -A3 "Pads MIDI"'

# b. external_pad_leds must be true, or the first pad touch wipes group colours
ssh root@192.168.2.123 'grep -o "external_pad_leds.*" /root/zynth/MaschineMK2_linux/maschine.json'

# c. The daemon is running and its watchdog cadence is sane (~8-40 s per reopen)
ssh root@192.168.2.123 'systemctl is-active maschine-mk2; journalctl --since "-10min" | grep -c "watchdog: input stalled"'

# d. Driver load lines visible while auditing
ssh root@192.168.2.123 'systemctl set-environment ZYNTHIAN_LOG_LEVEL=20; systemctl restart zynthian'
```

### Results — 2026-08-11

| Check | Result |
|---|---|
| a. Pads MIDI routing | **FAIL — two routes.** `Pads MIDI → ZynMidiRouter:dev3_in` **and** `→ ZynMidiRouter:dev2_in`. Both appeared after a clean boot (Pi up 14 min), so this is not the 2026-08-08 stale-`jack_connect` cause; suspect a device re-enumeration after a watchdog reopen giving the alias a second zmip slot. **Resolve before step 1** — a doubled route makes every CC arrive twice and would corrupt the whole audit |
| b. `external_pad_leds` | PASS — `"external_pad_leds": true` present in `/root/zynth/MaschineMK2_linux/maschine.json` |
| c. Daemon + watchdog | PASS — `maschine-mk2` active, reopens roughly one per 30-60 s against the ~8 s healthy baseline |
| d. Log level | `ZYNTHIAN_LOG_LEVEL` is **unset** — it did not survive the reboot. Re-set it before auditing, and unset it afterwards |

---

## Step 1 — Button audit `[pending — needs button presses]`

Find the port, then dump it:

```bash
ssh root@192.168.2.123
jack_lsp | grep -i pads
jack_midi_dump "a2j:maschine rs [129] (capture): Pads MIDI"
```

Press each button **once, alone**, and record what arrives. This settles which
physical pair emits CC 5/6 — the driver's old constant was named
`CC_PAGE_LEFT`, while `CLAUDE.md` calls that pair the arrows beside the display.
The owner's names are authoritative; the table below is what the driver now
assumes.

| Button | Owner's name | Assumed CC | Observed CC | Release event? | Notes |
|---|---|---|---|---|---|
| Arrows beside the display, left / right | **DL / DR** | 5 / 6 | | | Bound to page stepping |
| Master section, beside the big encoder | **ML / MR** | 13 / 14 | | | Bound to sound stepping |
| Transport ◀STEP / STEP▶ | **TL / TR** | 48 / 47 | | | Expected **swallowed** by the daemon for its own page indicators — confirm nothing arrives |
| CONTROL | | 11 | | | Mode |
| STEP | | 32 | | | Mode |
| ALL | | 38 | | | Mode |
| AUTO | | 37 | | | Mode — FILTER |
| VOLUME | | 51 | | | Mode — MIXER. **Needs the Task 10 patch deployed** |
| SWING | | 50 | | | Needs Task 10 |
| SHIFT | | 49 | | | Needs Task 10 |
| MUTE | | | | | Record whatever it sends |
| GRID | | | | | SP4 gesture |
| SELECT | | | | | |
| VIEW | | | | | |
| PAD MODE | | | | | Gated by SHIFT inside the daemon |
| NAVIGATE / NAV | | | | | |

Record the **physical location** of every button that emits, not only its label —
the panel silkscreen, the daemon's token names and the driver's constant names
have all disagreed with each other before.

---

## Step 2 — AUTO reachability `[pending — needs button presses]`

Confirm CC 37 reaches the driver rather than being swallowed the way 47/48 are.
The shipped daemon already has an `"auto"` arm, so this is expected to pass; it
is checked separately because FILTER mode is unusable if it does not.

- [ ] CC 37 appears in `jack_midi_dump` on press
- [ ] CC 37 appears on release
- [ ] The driver's FILTER mode LED lights

---

## Step 3 — Post-patch check `[pending — needs button presses]`

After deploying `MaschineMK2_linux` `39c4503` (Task 10):

- [ ] SHIFT emits CC 49 on press **and** release
- [ ] SWING emits CC 50 on press and release
- [ ] VOLUME emits CC 51 on press and release
- [ ] **PAD MODE still behaves.** SHIFT remains a live internal modifier — the
      `set_mod` block runs before the emit arm and gates PAD MODE and the B6
      encoder. If PAD MODE broke, the patch removed something it should not have

---

## Step 4 — Symbol audit `[done 2026-08-11]`

```bash
ssh root@192.168.2.123 'nm -D --defined-only /zynthian/zynthian-ui/zynlibs/zynmixer/build/libzynmixer.so | awk "\$2==\"T\"{print \$3}" | sort'
ssh root@192.168.2.123 'nm -D --defined-only /zynthian/zynthian-ui/zynlibs/zynseq/build/libzynseq.so  | awk "\$2==\"T\"{print \$3}" | sort'
```

### Mixer — the Pi runs an **older API than this checkout**

| Symbol | On the Pi | Note |
|---|---|---|
| `enableDpm` | **yes** | |
| `getDpm`, `getDpmHold`, `getDpmStates` | **yes** | All return **dBFS** (`convertToDBFS` in `mixer.c`) |
| `updateDpmStates` | **no** | This is what the WSL checkout's Python wrapper calls |

The Python side differs too, and this matters more than the C symbols:

| | WSL checkout | The Pi |
|---|---|---|
| Module | `zynlibs/zynmixer/zynmixer.py` | `zyngine/zynthian_engine_audio_mixer.py` |
| Read | `update_dpm_states()` fills `mixer.dpm`, a `DPM` array of `(a, b, a_hold, b_hold, mono)` | `get_dpm_states(start, end)` → `[[a, b, hold_a, hold_b, mono], …]` |
| Enable | `enable_dpm(enable)` | `enable_dpm(start, end, enable)` |

**Consequence, and it was silent:** Task 8's `_meter_frac` guarded on
`hasattr(mixer, "update_dpm_states")`, which is False on the Pi, so the mixer
meter would have degraded to fader position on the only machine that runs it —
with no error anywhere. `enable_dpm(True)` would additionally have raised
`TypeError` into a swallowing `except`. Fixed in `zynthian-ui` `f1c98493`: the
driver now tries the new API, falls back to the old one, and only then gives up.

- [x] Decision recorded: **the meter works on the Pi's API**, no feature is
      dropped. If a future Zynthian update moves to the newer wrapper, the same
      code takes the first branch and keeps working.

### Sequencer — everything pass two and SP2 need is present

`addNote` is `_ZN7Pattern7addNoteEjhhff` — **five arguments** (`step, note,
velocity, duration, offset`). Also present, all of which pass two or SP2 reach
for: `setPlayChance` / `getPlayChance`, `setNotePlayChance` /
`getNotePlayChance`, `setSwingAmount` / `setSwingDiv`, `setStutterCount` /
`setStutterDur`, `changeStutterCountAll` / `changeStutterDurAll`.

**RATCHET is unblocked** — `setStutterCount` exists on the installed library.

---

## Step 5 — SOLO gestures `[pending — needs button presses]`

The oldest unverified claim in the project. Know before pressing: `zynmixer`'s
solo is **additive, not exclusive**, and it has a special case at
`MAX_NUM_CHANNELS - 1` where the main strip clears every solo.

- [ ] Hold SOLO, press F1, then F3. Record: do **both** channels solo, or only
      the last pressed?
- [ ] Release SOLO. Record whether the solos clear (momentary) or persist
      (latched)
- [ ] Tap SOLO alone. Record whether the F row becomes solos
- [ ] Press the main strip's solo. Confirm it clears every solo

Whatever is observed is the specification — the surface behaviour was never
designed, only assumed.

---

## Deployment notes

These have all bitten before. None is optional.

**Moving commits to the Pi.** The Pi cannot fetch from GitHub — root has no
auth. Bundle on WSL, fetch on the Pi:

```bash
# WSL
cd ~/zynth/MaschineMK2_linux && git bundle create /tmp/mk2.bundle main
scp /tmp/mk2.bundle root@192.168.2.123:/tmp/

# Pi
cd /root/zynth/MaschineMK2_linux
git fetch /tmp/mk2.bundle main:refs/remotes/origin/main && echo FETCH_OK
```

**Check the fetch exit status.** A bare `git reset --hard origin/main` once
rewound the tree on the Pi because the fetch had silently failed.

**After any reset on the Pi**, re-set `"external_pad_leds": true` in
`/root/zynth/MaschineMK2_linux/maschine.json`. It is not in git there, and
without it the first pad touch destroys the driver's per-group colours.

**After any Zynthian update**, re-run
`~/zynth-docs/tools/patch-autoconnect-maschine.py`. Without it Zynthian never
gives the daemon's virtual port a zmip slot, the driver is "Found" but never
"Loaded", and the rig does nothing at all with no error.

**Before trusting any measurement**, re-check step 0a. `jack_lsp -c | grep -A3
"Pads MIDI"` must show exactly **one** `devN_in`.

**After the audit**, unset the log level:
`systemctl unset-environment ZYNTHIAN_LOG_LEVEL`.

---

## What G4 does not cover

G5 runs after deployment and is separate: DSP load mean and p95, xrun count,
segfault and traceback count, memory over twenty minutes, watchdog reopen
cadence against the ~8 s healthy baseline, plus a mixer-mode check that meter
quantisation really does stop the repaint storm — hold a page still on a silent
channel and watch the OSC volume.
