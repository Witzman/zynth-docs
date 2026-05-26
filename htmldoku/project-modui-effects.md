# Audio FX Chain with MOD-UI

**Goal:** Route ESI U46DJ mic input (CH 1/2) and line input (CH 3/4) through a MOD-UI LV2 effects pedalboard designed in the browser, with processed audio on ESI outputs.
**Prerequisites:** ESI U46DJ configured as Zynthian audio device at 44.1 kHz (see [ESI U46DJ Audio Setup](project-u46dj-audio-setup.html)). JACK running with four capture ports confirmed.
**Access:** SSH · Webconf · Browser (port 8888)

---

## Part 1 — Pass-Through: MOD-UI Chain with ESI Input `[draft]`

Add a MOD-UI chain, route ESI mic input (capture_1/2) through it, confirm unprocessed audio reaches ESI outputs. This proves the JACK routing before adding any effects.

### Step 1 — Install required LV2 plugins

Mic and line processing needs EQ, compression, and reverb plugins. Install them now so they are available in MOD-UI's browser:

```bash
ssh root@zynthian.local
apt install -y calf-plugins lsp-plugins
```

After install, regenerate the LV2 cache:

```bash
systemctl restart zynthian
```

Wait ~30 seconds, then reload `http://zynthian.local`.

**Verify:**

```bash
jalv --list | grep -i "calf\|lsp" | head -10
```

Expected: multiple Calf and LSP plugin URIs listed.

### Step 2 — Add a MOD-UI chain in Zynthian

In VNC, tap **+** (bottom-left) → **Audio Input** → **MOD-UI**.

Zynthian starts the MOD-UI engine and creates JACK audio ports for the pedalboard.

**Verify:** A MOD-UI chain appears in the main screen.

### Step 3 — Open the MOD-UI browser interface

Open `http://zynthian.local:8888` in a browser.

The MOD-UI pedalboard editor loads. You see a canvas with two fixed blocks on the left (**Audio Input**) and right (**Audio Output**).

**Verify:** MOD-UI pedalboard canvas loads with Audio Input and Audio Output blocks visible.

### Step 4 — Build a pass-through connection in the pedalboard

Connect the Audio Input directly to the Audio Output — no plugins yet. In MOD-UI:

1. Click the output port circle on the **Audio Input** block (right side, port 1).
2. Drag the cable to the input port on the **Audio Output** block (left side, port 1).
3. Repeat for port 2 (right channel).

**Verify:** Two cable connections run from Audio Input to Audio Output in the pedalboard canvas.

### Step 5 — Find MOD-UI JACK port names

```bash
ssh root@zynthian.local
jack_lsp | grep -i "mod\|modui\|mod-host"
```

Note the input port names — they look like `effect_1:in_l` and `effect_1:in_r` or `mod-monitor:in_l` / `mod-monitor:in_r`. The exact names depend on the pedalboard state.

Also note the ESI capture ports:

```bash
jack_lsp | grep capture
```

Expected: `system:capture_1` through `system:capture_4`.

**Verify:** You can identify both the MOD-UI input ports and the ESI capture ports.

### Step 6 — Connect ESI mic input to MOD-UI

Replace `mod-host:in_l` and `mod-host:in_r` with the actual MOD-UI input port names from Step 5:

```bash
jack_connect system:capture_1 mod-host:in_l
jack_connect system:capture_2 mod-host:in_r
```

**Verify:**

```bash
jack_lsp -c | grep -A2 "capture_1\|capture_2"
```

Expected: capture_1 and capture_2 each show a connection to a MOD-UI port.

### Step 7 — Set ESI input source to MIC

On the U46DJ front panel, set the **CH 1/2 selector** to **MIC**. Connect a microphone to the MIC input (balanced 1/4" jack, front panel).

Speak or make noise into the mic.

**Verify:** Audio from the mic is audible through the ESI Mix Out → monitors. Unprocessed pass-through confirms JACK routing is correct.

---

## Part 2 — Mic Chain: EQ + Compressor + Reverb `[draft]`

Build a mic processing chain inside MOD-UI. Signal path: Mic → LSP EQ → Calf Compressor → Calf Reverb → Output.

### Step 1 — Open MOD-UI at port 8888

Open `http://zynthian.local:8888`. The pass-through pedalboard from Part 1 should be loaded.

**Verify:** Pedalboard canvas loads with the two cable connections from Part 1.

### Step 2 — Add LSP Parametric EQ

In MOD-UI, click the **+** button (or plugin browser icon). Search for **LSP Parametric Equalizer** (or `lsp`). Select the **LSP Parametric Equalizer x16 Stereo** plugin.

Drag it onto the canvas between Audio Input and Audio Output.

**Verify:** EQ plugin block appears on the canvas.

### Step 3 — Wire the EQ

1. Delete the direct Audio Input → Audio Output connections (click the cable, press Delete).
2. Connect **Audio Input** port 1 → **EQ** left input.
3. Connect **Audio Input** port 2 → **EQ** right input.
4. Leave EQ outputs unconnected for now.

**Verify:** Audio Input connects into EQ.

### Step 4 — Add Calf Compressor

Click **+** → search **Calf Compressor**. Drag it onto the canvas to the right of the EQ.

Connect:
- **EQ** left output → **Calf Compressor** left input
- **EQ** right output → **Calf Compressor** right input

**Verify:** EQ feeds into Compressor on the canvas.

### Step 5 — Add Calf Reverb

Click **+** → search **Calf Reverb**. Drag it to the right of the Compressor.

Connect:
- **Calf Compressor** left output → **Calf Reverb** left input
- **Calf Compressor** right output → **Calf Reverb** right input

Then connect Reverb outputs to Audio Output:
- **Calf Reverb** left output → **Audio Output** port 1
- **Calf Reverb** right output → **Audio Output** port 2

**Verify:** Full chain: Audio Input → EQ → Compressor → Reverb → Audio Output.

### Step 6 — Configure EQ for mic

Click the **LSP Parametric EQ** block to open its controls. Basic mic EQ settings:

| Band | Type | Freq | Gain |
|------|------|------|------|
| 1 | High-pass | 80 Hz | — | Cuts low-end rumble |
| 2 | Bell | 200 Hz | −3 dB | Reduces muddiness |
| 3 | Bell | 3 kHz | +2 dB | Adds presence |

Apply via the MOD-UI parameter sliders. Adjustments are live — speak into the mic and listen.

**Verify:** EQ changes are audible in real time.

### Step 7 — Configure Calf Compressor

Click **Calf Compressor** to open controls. Starting settings for mic:

| Parameter | Value |
|-----------|-------|
| Threshold | −18 dB |
| Ratio | 4:1 |
| Attack | 10 ms |
| Release | 100 ms |
| Makeup Gain | +6 dB |

**Verify:** Loud mic peaks are reduced, quiet passages raised. Sound is more even in level.

### Step 8 — Configure Calf Reverb

Click **Calf Reverb** to open controls. Starting settings:

| Parameter | Value |
|-----------|-------|
| Room Size | 0.5 |
| Diffusion | 0.7 |
| Wet Amount | 0.25 |
| Dry Amount | 1.0 |

**Verify:** A subtle room reverb is audible on the mic signal. Dry signal (Dry Amount) remains dominant.

### Step 9 — Test full mic chain

Speak into the mic. The signal passes through EQ → Compressor → Reverb → ESI outputs.

**Verify:** Mic audio sounds processed — cleaner, more even in level, with a touch of reverb.

---

## Part 3 — Line Input: Second Processing Path `[draft]`

Add a second effects path in MOD-UI for the ESI line input (capture_3/4 — CH 3/4).

### Step 1 — Check MOD-UI audio input count

MOD-UI pedalboard exposes a fixed number of audio inputs to JACK. By default this may be stereo (2 inputs). To handle mic (2ch) and line (2ch) simultaneously, the pedalboard needs 4 audio inputs.

Check current MOD-UI port count:

```bash
ssh root@zynthian.local
jack_lsp | grep -i "mod\|mod-host"
```

If only 2 input ports appear, you need a second MOD-UI chain (Step 2 route) or to expand the input count (not yet supported cleanly via Zynthian UI).

**Verify:** Note how many input ports MOD-UI exposes.

### Step 2A — If MOD-UI has only 2 inputs: add a second chain

In VNC, tap **+** → **Audio Input** → **MOD-UI**. A second MOD-UI instance starts on a new JACK client.

Find its ports:

```bash
jack_lsp | grep -i mod
```

Connect ESI line input to the second MOD-UI instance:

```bash
jack_connect system:capture_3 <second-modui-in-l>
jack_connect system:capture_4 <second-modui-in-r>
```

Replace `<second-modui-in-l>` with the actual port name from `jack_lsp`.

Open `http://zynthian.local:8888` — the pedalboard editor may switch between instances. Use the MOD-UI instance selector (if shown) to work on the second chain.

**Verify:** Second MOD-UI chain is connected to capture_3 and capture_4.

### Step 2B — If MOD-UI has 4 inputs: add line path to same pedalboard

If 4 input ports are available, add a second signal path on the same pedalboard canvas using inputs 3 and 4:

In MOD-UI browser, add separate EQ + Compressor + Reverb plugins for the line path. Connect Audio Input ports 3/4 → line EQ → line Compressor → line Reverb → Audio Output ports 3/4 (or merge into 1/2 via a Mixer plugin).

### Step 3 — Set ESI input source to LINE

On the U46DJ front panel, set the **CH 3/4 selector** to **LINE**. Connect a line-level source (keyboard, drum machine, mixer output) to the CH 3/4 RCA inputs on the rear panel.

**Verify:** Audio from the line source is audible through monitors, processed by the second MOD-UI chain.

### Step 4 — Build line effects chain in MOD-UI

For a line-level source, a simpler chain works well. In the second MOD-UI instance or the second path:

- **LSP Parametric EQ** — gentle high-pass at 40 Hz, subtle tone shaping
- **Calf Compressor** — lighter settings (Ratio 2:1, Threshold −12 dB)
- **Calf Reverb** (optional) — or skip reverb if the source already has its own space

Wire and configure as in Part 2 Steps 2–8, adjusted for line levels.

**Verify:** Line input is processed independently from the mic chain.

### Step 5 — Save the MOD-UI pedalboard

In the MOD-UI browser, click the save icon (floppy disk) and name the pedalboard (e.g. `mic-line-processing`). MOD-UI saves pedalboards to `/zynthian/zynthian-my-data/mod-ui/pedalboards/`.

Then save a Zynthian snapshot: go to **Library → Snapshots**, type a name in the **Name:** field, click the checkmark button to save — the snapshot stores which MOD-UI chain(s) are loaded.

**Verify:** Pedalboard name appears in MOD-UI's saved pedalboard list. Snapshot saved in webconf.

---

## Going Further

- Add a gate (e.g. Calf Gate) before the compressor to suppress mic noise floor during silence
- Use MOD-UI's built-in MIDI control to automate effect parameters from SMC-PAD knobs
- Add a second reverb with longer tail for ambient mic treatment — blend two reverb plugins in parallel
- Route the processed output back through a Zynthian synth chain for parallel processing
- Install additional LV2 plugins for more effect options: `apt install swh-plugins` adds dozens of classic effects
