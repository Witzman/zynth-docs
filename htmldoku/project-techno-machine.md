# Techno Machine — Build the Rig

**Goal:** Reproduce the Techno Machine — an eight-channel generative groovebox played entirely from a Maschine MK2 — on your own Zynthian.
**Prerequisites:**

- **Maschine MK2 Controller** tutorial, Parts 1 and 3 verified. That page builds the Rust HID daemon, adds the udev rules, exports the daemon's port through `a2jmidid`, installs the JACK connect script and runs the daemon as a systemd service. This page does not repeat any of it.
- Raspberry Pi 4 with ZynthianOS, touchscreen, SSH access.
- The LV2 plugins the rig uses, already installed: `tap-echo.lv2` and `tap-reverb.lv2` (system path `/usr/lib/lv2`, from the `tap-plugins` package), plus `JC303.lv2`, `Obxd.lv2` and `padthv1.lv2` (Zynthian path `/zynthian/zynthian-plugins/lv2`).
- The SFZ drum machine kits at `/zynthian/zynthian-data/soundfonts/sfz/Drum Machines` — 42 kits on a stock install.

**Access:** SSH · touchscreen · webconf (LV2 cache only)

This is a **build** tutorial: it gets the instrument standing up and proves each layer works. How to *play* it — the euclidean drums, the Turing voices, LOCK, the performance gestures — is the separate user manual, linked under **Going Further**.

The order matters and is not the obvious one. The surface comes first, then the sound, then the space. Each part is verifiable on its own, and a fault in the later parts is much harder to read if you cannot trust the earlier ones.

---

## Part 1 — Make the surface talk to Zynthian `[draft]`

The Techno Machine is a **ctrldev driver** — Zynthian's plug-in point for a control surface. It is a Python class in `zyngine/ctrldev/` that Zynthian loads when it sees a matching MIDI device, and it reaches directly into the running instrument: the chain manager, the mixer, the sequencer library.

At the end of this part the Maschine draws its own screens and lights its own buttons with **no chains and no snapshot loaded at all**. That is the point of doing it first.

### Step 1 — Confirm the daemon is running

```bash
ssh root@192.168.2.123 'systemctl is-active maschine-mk2'
# → active
```

If it is not active, stop here and finish the **Maschine MK2 Controller** tutorial. Nothing below can work without it.

**Verify:** the MK2's two displays are lit and its buttons are not all dark. Blank displays plus dark buttons means the daemon is down.

### Step 2 — Turn on external pad LEDs

The driver owns the pad colours. If the daemon also repaints them, the first pad you touch destroys the per-channel picture.

Edit `maschine.json` in the daemon's working directory and add the flag:

```bash
ssh root@192.168.2.123 'grep external_pad_leds /root/zynth/MaschineMK2_linux/maschine.json'
# → "external_pad_leds": true
```

If the line is missing, add it inside the top-level object, then restart the daemon:

```bash
ssh root@192.168.2.123 'systemctl restart maschine-mk2'
```

**This flag is not in git.** A `git reset --hard` in the daemon's repo wipes it, so re-set it after every deploy that touches the daemon.

**Verify:** the flag is present and the daemon restarted without error in `journalctl -u maschine-mk2 -n 20`.

### Step 3 — Patch zynautoconnect

Zynthian only hands a **zmip slot** to ports it considers hardware MIDI sources. The daemon's port is a virtual `a2j` client, so without a patch Zynthian sees it, lists the driver as *Found*, and never *Loads* it. The rig then does nothing at all, with no error anywhere.

```bash
scp ~/zynth-docs/tools/patch-autoconnect-maschine.py root@192.168.2.123:/root/
ssh root@192.168.2.123 'python3 /root/patch-autoconnect-maschine.py'
# → zynautoconnect patched: whitelist + stable uid
```

The patch does two things: it adds `maschine rs.*Pads MIDI` to the source whitelist, and it pins a stable uid — `virtual:maschine.rs/Maschine MK2 Pads` — because the ALSA client number embedded in the port name changes across boots and a ctrldev driver binds by device id.

**Re-run this after every Zynthian system update.** An update replaces `zynthian_autoconnect.py` and silently takes the binding with it. The script is idempotent and says `already patched, nothing to do` when it has nothing to do.

**Verify:** the script prints the patched message, or reports it was already patched.

### Step 4 — Copy the three driver files

```bash
cd ~/zynth/zynthian-ui/zyngine/ctrldev
scp zynthian_ctrldev_maschine_mk2.py techno_lib.py maschine_mk2_lib.py \
    root@192.168.2.123:/zynthian/zynthian-ui/zyngine/ctrldev/
```

Three files, and all three are required:

| File | What it is |
|---|---|
| `zynthian_ctrldev_maschine_mk2.py` | The driver. Claims the pad port exclusively (`unroute_from_chains = True`) so pads never reach a chain by themselves. |
| `techno_lib.py` | All generative and presentational logic, with no Zynthian imports and no I/O. This is the unit-tested half. |
| `maschine_mk2_lib.py` | The screen framebuffer and the LED report layout. |

**Copy files, never use git on the Pi.** The Pi's `zynthian-ui` runs an upstream branch with these three files as untracked drop-ins; a `git reset --hard` or a bundle checkout there deletes them.

The driver manager globs **every** `*.py` in that directory and reads a `dev_ids` attribute off each one. `techno_lib.py` and `maschine_mk2_lib.py` therefore carry `dev_ids = []` even though they are helpers. Remove it from either and the whole Zynthian UI crash-loops every 14 seconds.

**Verify:** all three files are present in `/zynthian/zynthian-ui/zyngine/ctrldev/`.

### Step 5 — Restart in the right order: daemon first, UI second

```bash
ssh root@192.168.2.123 'systemctl restart maschine-mk2 && sleep 8 && systemctl restart zynthian'
```

Restarting the UI alone is fine. Restarting the **daemon** alone is not: `a2j` re-registers the Pads port onto a *new* zmip slot while the ctrldev driver stays bound to the dead one. The rig goes silent with no error, and it leaves a second stale route behind.

**Verify:** Zynthian's UI comes back on the touchscreen.

### Step 6 — Confirm the driver is Loaded, not merely Found

```bash
ssh root@192.168.2.123 'journalctl -u zynthian --since -3min | grep -i ctrldev'
```

You want a line saying the driver was **Loaded**. *Found* on its own means Step 3 did not take effect.

Then check the route is unique:

```bash
ssh root@192.168.2.123 'jack_lsp -c | grep -A3 "Pads MIDI"'
```

Exactly **one** `ZynMidiRouter:devN_in` connection. Two means a stale manual `jack_connect` from an earlier session is still alive — `zynautoconnect` only tears down connections it made itself, and `jackd` outlives a Zynthian restart. Extra routes make every pad tap fire twice.

**Verify:** the MK2's left display draws the tab row `A KICK`, `B SNAR`, `C CLAP`, `D CHAT` and the right draws `E OHAT`, `F BASS`, `G LEAD`, `H PADS`, with a dotted rule and four encoder columns under each. The Group buttons light in their channel colours — warm red through green for the five drums, blue through cyan for the three voices. The **CONTROL** button is lit. No chains exist yet, so the channels sit at a flat mid brightness and nothing makes sound.

---

## Part 2 — The eight channels `[draft]`

The instrument is eight always-alive channels. Nothing is created, browsed for or torn down while you play, which is why the whole thing is a prepared snapshot rather than run-time construction.

Build them on the touchscreen. The driver's channel table is fixed, and **the MIDI channel is the contract** — the driver resolves a channel to a chain through `midi_chan_2_chain_ids`, not by chain order or title.

| Group | Driver name | Kind | Engine | MIDI channel |
|---|---|---|---|---|
| A | KICK | drum | LinuxSampler, an SFZ kit | 1 |
| B | SNAR | drum | LinuxSampler | 2 |
| C | CLAP | drum | LinuxSampler | 3 |
| D | CHAT | drum | LinuxSampler | 4 |
| E | OHAT | drum | LinuxSampler | 5 |
| F | BASS | voice | JC303 | 6 |
| G | LEAD | voice | Obxd | 7 |
| H | PADS | voice | padthv1 | 8 |

### Step 1 — Add the five drum chains

In the touchscreen mixer, tap **+** to reach the Add Chain screen, then tap **Instrument**. Pick LinuxSampler, then a kit from the **Drum Machines** bank.

Set each chain's MIDI channel in **Chain Options**. Chains one through five take MIDI channels 1 to 5.

Chain titles are cosmetic — the shipped snapshot uses `Kick`, `Snare`, `Clap`, `Closed Hat`, `Open Hat` — because the four-character names on the MK2's display come from the driver's own table, not from the chain.

**Verify:** the touchscreen mixer shows five strips. On the MK2, the tab of a built channel is no longer at flat mid brightness.

### Step 2 — Add the three voice chains

Same route — **+** → **Instrument** → LV2 Plugin — for `JC303`, `Obxd` and `padthv1`, on MIDI channels 6, 7 and 8 in that order.

These three were chosen because all three publish the four control ports the voice CONTROL page needs, measured off live chains rather than read from a config flag: cutoff, resonance, filter-envelope amount and a decay-shaped control.

**Verify:** the mixer shows eight strips plus main. Play notes into each voice from any MIDI keyboard and hear it.

### Step 3 — Set the gain staging

Set every channel strip to **0.19** and main to **0.80** on the touchscreen mixer.

That is not arbitrary and it is not conservative. One sampler channel peaks at 1.24 before the mixer, and eight of them summed to **2.92** on the main bus — nearly three times full scale. The sampler's own volume control is not the fix; taking it from 96 to 40 moved the bus peak by about 1.5 dB. The mixer strips are the fix, and main at 0.80 leaves the MASTER knob travel in both directions.

The reverb and delay you add in Part 3 both pass **dry at unity**, as an insert must, so they add level on top of this rather than replacing it.

**Verify:** eight strips near the bottom of their travel, main a little below the top.

### Step 4 — Save the snapshot

On the **touchscreen**: open Snapshots, go **into** bank `000`, and use the first entry, **Save as new snapshot**. The shipped name is `016-techno_maschine`.

> **Do not save from webconf's Snapshots page.** Its **Name:** field plus the checkmark icon **renames the selected bank**. It has destroyed bank `000` once already. A snapshot saved to the snapshots root rather than into a bank is invisible in the UI.

**Verify:** the snapshot appears inside bank `000` and reloads without error.

### Step 5 — Prove the generator writes

On the MK2, press **STEP**, select Group **A**, and turn encoder 1 (`HITS`) to 4. Press **Play**.

The driver writes the euclidean pattern into Zynthian's own sequencer, so the pads, the touchscreen pattern editor and what you hear are three views of one truth. A white pad sweeps the grid as the playhead moves.

**Verify:** four-on-the-floor from Group A, four bright pads on the beat, one white pad sweeping. Select **F** and the phrase from the BASS voice changes shape every cycle or so.

---

## Part 3 — The sixteen post-fader inserts `[draft]`

Each of the eight channels carries its own reverb and its own delay — sixteen plugin instances. They are **inserts fed from the mixer strip's output**, verified on the wire:

```
LinuxSampler / synth → zynmixer strip (fader, pan, mute) → TAP Stereo Echo → TAP Reverberator → main
```

Post-fader is what makes them behave: an insert fed from the strip's output inherits the channel's fader **and its mute**, so muting a channel kills its reverb and delay tail with it instead of letting it ring out.

Both plugins were chosen for one measured property: their wet control is a **true wet level**, not a dry/wet crossfade. Sweep the wet to maximum and the dry is still there at exactly the same level. That is what lets encoders 7 and 8 behave like sends on every channel forever. Every cheap candidate — MDA Ambience, MDA DubDelay, CAPS PlateX2, MDA Delay, `lcrDelay`, `bolliedelay` — turned out to be a crossfade and was rejected. The cheapest true sends, `gverb` and MaGigaverb, are **mono in**, and an insert placed after the strip's pan would collapse every channel to centre.

Building sixteen instances by hand is sixteen chances to get one wrong, so build **one channel** and let a script replicate it.

### Step 1 — Add both inserts to one channel

Select the Kick chain. In **Chain Options** → **Add Audio-FX processor** → **LV2 Plugin**, add **TAP Stereo Echo**. Repeat for **TAP Reverberator**.

Order matters: echo first, reverb second, so the reverb hears the echo's repeats. That is the order the shipped snapshot has on the wire.

**Verify:** the chain graph shows the sampler, then TAP Stereo Echo, then TAP Reverberator.

### Step 2 — Force dry to unity and wet to off

Open each insert's control screen and set:

| Plugin | Control | Value | Why |
|---|---|---|---|
| TAP Stereo Echo | `dryLevel` | **0.0 dB** | Ships at −4 dB, which quietly costs every channel about 8 dB across the pair |
| TAP Stereo Echo | `lecholevel`, `recholevel` | **−70.0 dB** | Wet starts off; encoder 8 opens it |
| TAP Reverberator | `drylevel` | **0.0 dB** | Same reason |
| TAP Reverberator | `wetlevel` | **−70.0 dB** | Wet starts off; encoder 7 opens it |

A default that happens to work is still a default. One candidate reverb defaults its dry port to near zero.

**Verify:** the channel sounds exactly as loud as it did before you added the inserts.

### Step 3 — Save, then clone the pair onto the other seven chains

Save the snapshot again from the touchscreen, then:

```bash
scp ~/zynth-docs/tools/build-techno-snapshot.py root@192.168.2.123:/root/
ssh root@192.168.2.123 'python3 /root/build-techno-snapshot.py'
```

The script works offline on the `.zss` JSON: it finds the chain that carries both inserts, then appends the same two processors to every other chain with a MIDI channel, giving each a fresh processor id and copying the template's `fader_pos`. It backs the file up to `…​.zss.bak` first.

It deliberately does **not** build the whole snapshot. There is no CUIA that executes code, so a script cannot reach Zynthian's live state manager from outside the UI process — and hand-maintaining `fader_pos` is exactly the kind of guess this project does not make. One channel is built by hand so that Zynthian, not the script, decides slots and processor state.

**Verify:** the script prints `inserts cloned` for seven chains and skips the template.

### Step 4 — Reload and re-save

Load the snapshot on the touchscreen, then save it again. What ends up on disk is then Zynthian's own output rather than the script's.

**Verify:**

```bash
ssh root@192.168.2.123 'jack_lsp | grep -c TAP'
# → 64      (16 instances x 4 ports)
```

Then, on the MK2, press **CONTROL**, select any channel and sweep encoder 7 (`REVERB`) from 0 to 100. The dry signal must still be there, at the same level, at the top of the sweep. If it fades out as the wet comes in, you have a crossfade plugin, not a wet level, and the instrument's whole send contract is broken.

Expect the wet knobs to feel back-heavy: 0-100 maps onto −70 dB … +10 dB, so 25 is inaudible, 50 is barely there, 88 equals dry and the musically useful travel is roughly **60 to 100**.

---

## Part 4 — Commission the surface `[draft]`

Everything is standing. This part walks the surface once so that the first fault you meet on stage is not also the first time you have used the control.

### Step 1 — The five modes

Exactly one mode is lit at any moment, and CONTROL is home. Pressing a lit mode button returns to CONTROL.

| Button on the panel | Mode | What the eight encoders become |
|---|---|---|
| **CONTROL** | CONTROL | What the selected channel *sounds like*. Encoders 6, 7, 8 are LEVEL, REVERB, DELAY on every channel of both kinds — the one absolute muscle memory in the machine. |
| **STEP** | STEP | What the selected channel *plays*. Euclid on a drum; the Turing register on a voice. |
| **ALL** | ALL | The machine's globals: ROOT, SCALE, BPM, MASTER, and the four ganged space parameters. |
| **VOLUME** | MIXER | One verb across all eight channels: LEVEL, then REVERB, then DELAY on the next pages. |
| **AUTO** | FILTER | CUTOFF and RESO across all eight channels. |

**Verify:** each button lights alone, and the page label on the display changes with it.

### Step 2 — The page rings

The two arrows **beside the display** step through the pages *within* the current mode, wrapping. Each mode remembers its own page, per channel kind.

| Mode | Pages |
|---|---|
| CONTROL | one page (plus generated pages built from whatever ports the chain publishes) |
| STEP, drum | STEP · SWING spread · CHANCE spread |
| STEP, voice | STEP · SWING spread · CHANCE spread · DENSITY spread |
| ALL | one page (plus generated pages) |
| MIXER | LEVEL · REVERB · DELAY |
| FILTER | CUTOFF · RESO |

**Verify:** on STEP with a voice selected, the arrows walk four pages and come back around.

### Step 3 — Read the FILTER page honestly

On the FILTER page, the **five drum columns are dead** and show `----`. This is correct, not a fault: the driver resolves CUTOFF and RESO through a table of measured control symbols, LinuxSampler publishes **no controllers at all**, so a drum channel has nothing behind those knobs and the driver leaves them dead rather than silently moving something else.

FILTER therefore moves the three voices, on encoders 6, 7 and 8.

**Verify:** encoders 1-5 do nothing on the FILTER page; 6, 7 and 8 sweep the BASS, LEAD and PADS filters.

### Step 4 — Pads are the instrument

In every mode except STEP, the pads **play** the selected channel. On a drum all sixteen pads play that channel's own sound, because a drum channel *is* one sound. On a voice they play pitches.

STEP stays the step editor: there a pad toggles a step.

**Verify:** on CONTROL, tapping pads sounds the selected channel. Press STEP and the same pads edit the pattern instead.

### Step 5 — Overdub with REC

Hold **REC** and play. Notes land in the same pattern the generator writes, quantised to the nearest step, and the note's length is **how long you held the pad**.

Recording **adds**, it does not overwrite: the generated line freezes and your note goes on top, so a recorded step can hold two notes. A played-in step lights **amber**.

Capturing a note makes the **player** the owner of that channel's pattern, and the generator stops writing it. Two routes hand it back, both destructive:

- **ERASE + Group** on that channel.
- Turning any knob that rewrites the pattern — on a drum HITS, ROTATE or DIV; on a voice LENGTH, DIV or RANDOM. LENGTH on a drum is deliberately excluded, because shortening a pattern preserves the steps that fit.

**Verify:** hold REC, tap a few pads on a voice, release. The pads you played light amber and repeat every cycle. Turn RANDOM and the take is replaced by the generator's line.

### Step 6 — Switch a channel's behaviour

**SHIFT + GRID** switches the selected channel between drum and voice behaviour. The **engine is not swapped** — a drum kit gets the Turing generator, and a synth gets euclid.

Two consequences worth knowing before you use it live:

- On a sampler the register walks the **kit's own note list**, not ROOT and SCALE, because there a note selects a sample and scale quantising would land most steps on empty keys.
- Euclid on a synth is a **root pulse**: ROOT plus that channel's OCTAVE.

Each kind keeps its own remembered parameters, but `div` and `beats` deliberately do not move — they are pattern time, not kind, and the groove must not jump on a switch. Hand-edited steps do not survive a switch, for the same reason turning HITS wipes them: switching rewrites the pattern from the generator.

A voice switched to drum may come up silent, because its `hits` can read back as 0. Its tab draws **dashed** to say so. Turn HITS up.

**Verify:** SHIFT + GRID on a drum channel, then STEP — the page now shows the voice columns (LENGTH, DIVIDE, RANDOM, GATE…). SHIFT + GRID again returns it, and the override clears itself.

### Step 7 — The performance gestures

| Gesture | Effect |
|---|---|
| **Group A-H** | Select the channel. Pads, tabs and all eight encoders follow. |
| **F1-F8** | Mute channel A-H regardless of which is selected. **Tap** latches, **hold** past 250 ms is momentary. The mute is on the mixer strip, so it is saved in the snapshot, visible on the touchscreen, and it cuts the FX tail too. |
| **SOLO** held + F | Momentary solo. Solo is **additive**, not exclusive. |
| **SOLO** tapped | Latches solo mode: the F row *is* solo until you tap SOLO again. |
| **ERASE** held + pad | Clears that step. A bare ERASE press does nothing at all. |
| **ERASE** held + Group | Silences that channel — HITS → 0 on a drum, play chance → 0 on a voice, which toggles back on a second press. It sets the *generator* to silence rather than wiping the note list, because a wiped list is written straight back by the next generator move. |
| **Play** | Start or stop all eight sequences together. |
| **Restart** | Every channel jumps to step 0 without stopping. |
| **DUPLICATE** | Undo for a voice: restore the previous Turing register, force RANDOM to 0, rewrite now. Up to four deep. Nothing on a drum. |
| Arrows in the **master section** | Step the selected channel's sound: a sample within the kit on a drum, an engine preset on a voice. |

**Verify:** F3 mutes CLAP from any selection and its Group button goes dark. ERASE alone does nothing.

### Step 8 — Health check

```bash
ssh root@192.168.2.123 'journalctl -u zynthian --since -3min | grep -iE "ctrldev|maschine|traceback|error"'
ssh root@192.168.2.123 'jack_lsp | grep -c TAP'                                  # 64
ssh root@192.168.2.123 'jack_lsp -c | grep -A3 "Pads MIDI"'                      # exactly one devN_in
ssh root@192.168.2.123 'journalctl --since -20min | grep -c "watchdog: input stalled, reopened"'
```

The watchdog line is **healthy**. The MK2's input dies after a few seconds under a kernel hidraw fault, and the daemon closes and reopens the device to recover; one reopen per eight seconds or so is the normal baseline. Much more often than that is a regression.

**Verify:** no tracebacks, 64 TAP ports, one route.

### Step 9 — Save the commissioned state

Save the snapshot once more from the touchscreen. The driver's own state — every voice's Turing register, its four-deep undo ring, the globals, the current mode and selected channel, which channels the player owns — rides inside the snapshot under the MIDI device's `ctrldev_state`.

**The registers are the part that matters.** A snapshot without them restores a machine that plays *different music*. With them, a locked voice comes back playing the exact line it had.

Press **Play** once after any load. Restoring a snapshot rewrites every sequence's play mode from the file, and a loop-all sequence shorter than the bar goes silent until the next bar sync; the driver re-forces LOOP on every transport start.

**Verify:** reload the snapshot and the instrument comes back playing the same music, with the same mode lit and the same channel selected.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Buttons do nothing, displays blank, Zynthian is up | The driver is **Found but never Loaded** — no zmip slot. Happens after any Zynthian update. | Re-run Part 1 Step 3, restart daemon then UI. |
| First pad touch destroys the pad colours | `external_pad_leds` missing from `maschine.json`. | Part 1 Step 2. Re-set it after any daemon deploy — it is not in git. |
| Rig goes silent after restarting the daemon alone | `a2j` re-registered the port on a new zmip slot; the driver is bound to the dead one. | Restart the UI too, then check the route is unique. |
| Phantom extra sounds on every pad tap | A stale JACK route from an earlier session. | `jack_lsp -c \| grep -A3 "Pads MIDI"` must show exactly **one** `devN_in`. Disconnect the extra. |
| A channel is silent, then returns on the next bar | LOOP play mode was rewritten from the `.zss`. | Press **Play** — LOOP is re-forced on every transport start. |
| A channel is silent and looks healthy | On a drum, HITS 0. On a voice, play chance 0 — set by ERASE + Group. Either way the tab draws **dashed**. | Turn HITS up, or press ERASE + that Group again. |
| Encoders feel dead or pinned at one end | An encoder ran into the daemon's 0-127 clamp. | Select another channel or change mode — both re-park every encoder at mid-travel. |
| The whole Zynthian UI dies, exit 139 | Unsynchronised access to the sequencer library from several threads. Every path in this driver holds one lock, so this is a regression. | Restart Zynthian and report it. It took 95 seconds to appear last time, so a short test will not reproduce it. |
| Mix distorts with several wets open | Both inserts pass dry at unity and the wets add on top. | Pull MASTER or the strips down. Design headroom is main 0.80 with strips 0.19. |

---

## Two divergences from the user manual

Recorded here because the manual is dated 2026-08-10 and the rig has moved since.

- **The manual's signal-flow diagram shows the reverb before the echo.** On the wire it is the other way round: strip → TAP Stereo Echo → TAP Reverberator → main. This page follows the wire.
- **The manual describes three pages, and pads that only toggle steps.** The shipped surface has five modes with page rings, pads as the instrument, REC overdub and SHIFT + GRID kind switching. Part 4 above is the current surface.

---

## Going Further

- **The user manual** — `docs/superpowers/techno-machine/2026-08-10-techno-machine-manual.md`. Everything about *playing* the instrument: the euclidean model, the Turing machine and LOCK, the reachable pattern lengths per division, gain staging, the LED language.
- **Re-measure on your own soundcard.** Every absolute figure in this project was taken on the Pi's built-in headphone output at 48 kHz. Relative plugin costs are card-independent; headroom figures are not.
- **A shared reverb and delay bus** is not possible in this shape. Zynthian's mixer has sixteen usable strips compiled in, and a correct send-tap topology needs twenty-six. Sixteen post-fader inserts is the answer to that constraint, not a preference.
- **Per-channel FX character** — reverb and delay parameters are ganged across all eight instances; only the wets differ. Identical character in eight boxes is most of the way to a coherent space.
