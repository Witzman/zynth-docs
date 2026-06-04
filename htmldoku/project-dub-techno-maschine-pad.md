# Dub Techno Live Rig — Maschine Pad Layer

**Goal:** Add the Maschine MK2 step sequencer as a live pad/chord layer over the Dub Techno drum and bass foundation, with dub-style delay and reverb, and documented live performance techniques.
**Prerequisites:**
- [Dub Techno Performance Loop](project-dub-techno-loop.html) Part 1 verified — `dub-techno-p1` snapshot loads and plays drums + bass
- [Maschine MK2 Step Sequencer](project-maschine-step-sequencer.html) Part 1 verified — sequencer confirmed firing MIDI on channel 2
**Access:** SSH · VNC · Webconf

---

## Part 1 — Pad Layer Over Drum+Bass `[draft]`

The Maschine step sequencer runs on its own internal clock — independent of Zynthian's transport. Over long periods it will drift relative to the drum and bass patterns. At dub techno tempos this drift is slow and the reverb wash of the pad layer masks it. For a 4–8 bar performance window, drift is negligible.

### Step 1 — Load the dub techno snapshot

In webconf, go to **Library → Snapshots** and load **dub-techno-p1**.

**Verify:** Mixer shows the FluidSynth drum chain and ZynAddSubFX bass chain.

### Step 2 — Confirm the Maschine daemon is running

```bash
ssh root@zynthian.local
systemctl status maschine-mk2.service --no-pager
```

Expected: `Active: active (running)`

If not running:

```bash
systemctl start maschine-mk2.service
```

**Verify:** Service is active.

### Step 3 — Add a pad chain on MIDI channel 2

In the Zynthian VNC desktop, tap **+** in the mixer.

1. Tap **Instrument**
2. Select **ZynAddSubFX**
3. Load a pad or string preset — a long-attack, sustained sound works best for dub techno
4. Open **Chain Options** for the new chain
5. Set **MIDI Channel** to **2**

[low] Suitable ZynAddSubFX pad preset names need Pi verification.

**Verify:** Three chains in the mixer — drums, bass, and a pad on channel 2.

### Step 4 — Match Maschine tempo to Zynthian BPM

The Maschine sequencer speed is set in milliseconds per half-step. Use this formula to match Zynthian's BPM:

```
speed_ms = 7500 / BPM
```

At 120 BPM: `speed_ms = 62`

[low] Speed adjustment via Shift + encoder B6 is marked "under construction" in the daemon — verify it accepts input and updates step rate before relying on this step.

If the encoder does not work, the default speed (100ms ≈ 75 BPM 16th notes) is still usable — drift will be more audible but the reverb tail masks it at low speeds.

**Verify:** After adjusting, step rate feels roughly aligned with the drum pattern.

### Step 5 — Program a pad pattern on Maschine

Enter sequencer mode on the Maschine MK2: hold **Shift**, press **Pad Mode**, release, hold **Shift**, press **Pad Mode** again.

Press **Group D** to set note base to C4 (middle C).

Toggle 3–5 pads to activate steps — sparse patterns suit dub techno. Example: steps 1, 5, 13.

Press **Play** on the Maschine MK2.

**Verify:** Pad chain plays notes at the programmed steps. No sound from drums yet.

### Step 6 — Start Zynthian transport

In VNC, open the Launcher (**PAD/STEP** on the V5 keypad) and tap **PLAY (▶)**.

**Verify:** Drum pattern and bass pattern play from Zynthian. Maschine pad layer plays simultaneously. All three layers audible together.

### Step 7 — Save the combined snapshot

In webconf, go to **Library → Snapshots**. Type `dub-techno-maschine-p1` in the **Name:** field and click the checkmark icon.

**Verify:** Snapshot saved. Reloading it restores the three chains. (The Maschine daemon and its step pattern are not saved in the snapshot — restart the daemon and reprogram the pattern after loading.)

---

## Part 2 — Dub Delay and Reverb `[draft]`

Add dub-style delay and reverb to the pad chain for the classic smeared, spacious dub techno texture.

### Step 1 — Add a delay effect to the pad chain

[low] Exact LV2 plugin name and chain insertion method need Pi verification. The following describes the expected workflow.

In VNC, tap the pad chain in the mixer to open it. Add an LV2 effect to the chain's FX slot — look for a tape delay or simple delay plugin.

Set:
- Delay time: aligned to the BPM (e.g. 500ms at 120 BPM for quarter-note delay)
- Feedback: 40–60%
- Wet/dry: 30–50%

[low] Delay time formula: `delay_ms = 60000 / BPM` for quarter note, `/ (BPM × 2)` for eighth note.

**Verify:** Pad notes leave a repeating echo tail.

### Step 2 — Add reverb to the pad chain

Add a reverb LV2 plugin after the delay in the pad chain.

Set:
- Room size: large (0.7–0.9)
- Decay: long (2–4s)
- Wet: 40–60%

**Verify:** Pad notes bloom into a long reverb wash. Combined with delay, the pad smears across the mix in a dub style.

### Step 3 — Save the effects snapshot

In webconf, save as `dub-techno-maschine-p2`.

**Verify:** Snapshot restores pad chain with delay and reverb settings intact.

---

## Part 3 — Live Performance Techniques `[draft]`

With the rig running, the Maschine pads become a live performance instrument. These are documented moves — repeatable techniques for building and releasing tension in a dub techno set.

### The Echo Throw

Toggle a step off while the sequencer is running. The delay and reverb tail continues after the note stops — the sound hangs in space and decays. Toggle it back on to bring it back.

Use this on the most harmonically rich step for a classic dub throw effect.

**Verify:** Toggling a step off leaves a decaying reverb tail. Re-enabling brings the note back cleanly on the next pass.

### The Dropout

Toggle all active steps off in quick succession. The reverb and delay wash sustains for 2–4 seconds before decaying to silence — a tension moment. Toggle steps back on to bring the pad layer back.

**Verify:** Full dropout + re-entry works cleanly without clicks or stuck notes.

### Group Transpose

Press a Group button (A–H) during playback to shift all step notes by octave. Use this to move the harmonic content up or down for a section change.

| Button | Note base | Character |
|--------|-----------|-----------|
| C | C3 | dark, low |
| D | C4 (default) | mid, warm |
| E | C5 | bright, airy |

**Verify:** Group button press shifts pad pitch while drums and bass continue unchanged.

### Resync After Drift

After extended play, the Maschine pad will drift out of phase with the drum and bass. To resync:

1. Press **Erase** on the Maschine to stop the sequencer
2. Listen for the downbeat of the Zynthian drum pattern
3. Press **Play** on the Maschine on the downbeat

**Verify:** Maschine pad realigns with the drum kick on step 1.

---

## Going Further

- Implement MIDI clock sync in the MaschineMK2_linux daemon — the daemon currently has no MIDI input; adding a MIDI clock receiver would let Zynthian drive Maschine's step rate automatically, eliminating drift
- Use two pad chains — one on channel 2 for Maschine, one on channel 1 for live pad playing with the Maschine pads in normal mode
- Combine with the SMC-PAD Mute Control tutorial to add per-chain muting to the drum and bass while performing with Maschine
- Program multiple patterns across sessions: document step layouts and Group button states as named "patches" in the snapshot notes
