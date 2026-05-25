# EMU Xboard CC Knob Mapping

**Goal:** Use the EMU Xboard's 16 CC knobs to control Zynthian synth parameters — two variants: static binding to one specific chain, and follow-channel binding where the knobs automatically control whichever chain matches the current MIDI channel.
**Prerequisites:** Zynthian running with the four-chain MIDI channel setup from [MIDI Channel Routing](project-midi-channel-routing.html). EMU Xboard connected via USB.
**Access:** VNC · SSH (for monitoring only)

---

## Background: Two Binding Variants

**Static binding** — a CC knob is bound to a parameter on one specific chain. The knob controls that chain regardless of what other chains exist. Use this when you always want a dedicated physical knob for one sound (e.g. knob 1 always sweeps the piano's filter).

**Follow-channel binding** — the same CC knob is bound to the same parameter type on every chain. Because Zynthian routes MIDI by channel, the knob automatically controls whichever chain the EMU is currently transmitting on. Switch channels, the knob follows.

---

## How CC Learn Works in VNC

The standard workflow uses the V5 hardware encoders. In VNC without hardware, the equivalent is:

1. Navigate to the control screen for the chain.
2. Tap the correct parameter page in the right-side page list.
3. **Click and hold** the target parameter knob for ~600ms. The knob border turns orange.
4. Turn an EMU CC knob. The CC number is captured immediately.
5. The orange highlight disappears. A CC badge appears under the knob name.

> **Note:** Click-and-hold triggering the learn mode needs Pi verification — this is the expected VNC equivalent of the long-press encoder. If holding the click does not trigger orange, try a firm single click on the knob and report back.

To cancel learn without assigning: click-hold the orange knob again, or tap Back.

---

## Part 1 — Static Binding to Chain 2 `[draft]`

Bind three EMU knobs permanently to Chain 2 (ZynAddSubFX strings). These knobs will control Chain 2 only — when the EMU is on channel 2.

Chain 2 uses ZynAddSubFX which has clearly labelled filter and envelope pages, making it ideal for the first demo.

### Step 1 — Open Chain 2 control screen

In VNC, tap **Chain 2** in the main screen. The chain control view opens. Tap the **control** subscreen if it is not already showing — the 4-knob display with the page list on the right.

### Step 2 — Bind knob 1 to Volume

Tap the **Global** page in the right-side page list. The 4 knobs change to show global parameters including **Volume**.

Click and hold the **Volume** knob for ~600ms. The border turns orange.

Turn **EMU knob 1**. The CC number is captured and appears as a badge on the Volume knob.

### Step 3 — Bind knob 2 to Filter Cutoff

Tap the **Filter** page in the right-side page list. The knobs now show filter parameters: Cutoff, Resonance, and others.

Click and hold the **Cutoff** knob for ~600ms → orange. Turn **EMU knob 2**. Captured.

### Step 4 — Bind knob 3 to Filter Resonance

Still on the Filter page. Click and hold **Resonance** → orange. Turn **EMU knob 3**. Captured.

### Step 5 — Test the bindings

Switch the EMU to **channel 2** (MIDI Channel button → data slider → 2).

Turn knobs 1, 2, and 3. Confirm:
- Knob 1 → Chain 2 volume changes (visible on the knob display and audible)
- Knob 2 → Filter sweeps open and closed
- Knob 3 → Resonance peaks and softens

Now switch the EMU to **channel 1**. Turn the same three knobs.

**Verify:** Knobs have no effect on channel 1 — Chain 1 is not bound to these CCs.

---

## Part 2 — Follow-Channel Binding `[draft]`

Bind the same three EMU knobs to Volume, Cutoff, and Resonance on all four chains. Because Zynthian routes MIDI by channel, the knobs will automatically follow the active MIDI channel.

Chain 4 uses setBfree (organ), which has drawbars instead of a filter. For that chain, Cutoff and Resonance will be substituted with drawbar controls — the concept is the same.

### Step 1 — Bind knobs to Chain 1

Chain 1 is FluidSynth. Tap **Chain 1** → control screen.

Tap the **Global** or first available page. Find **Volume**. Click-hold → turn **knob 1** → captured.

Navigate to a page with **Cutoff** (FluidSynth may label this differently — look for Filter or Cutoff across the available pages). Click-hold → turn **knob 2** → captured.

Find **Resonance** (or the closest equivalent). Click-hold → turn **knob 3** → captured.

### Step 2 — Bind same knobs to Chain 3

Chain 3 is ZynAddSubFX with a brass or lead preset.

Tap **Chain 3** → control screen → **Global** page → click-hold **Volume** → turn **knob 1** (same physical knob as Chain 1) → captured.

**Filter** page → click-hold **Cutoff** → turn **knob 2** → captured.

**Filter** page → click-hold **Resonance** → turn **knob 3** → captured.

### Step 3 — Bind knobs to Chain 4 (setBfree)

Chain 4 is setBfree. The control screen shows the drawbar widget instead of a standard filter page. Bind the three knobs to drawbars or expressive controls instead:

- Tap the **setBfree** page containing drawbar levels.
- Click-hold **Drawbar 1** (16' fundamental) → turn **knob 1** → captured.
- Click-hold **Drawbar 3** (8' principal) → turn **knob 2** → captured.
- Click-hold **Expression** (swell pedal level) → turn **knob 3** → captured.

### Step 4 — Test follow-channel behavior

Cycle the EMU through channels 1, 2, 3, 4. On each channel, turn knobs 1–3 and confirm:

- The knobs control the engine on that channel
- Switching channels switches which chain responds
- No other chain moves

**Verify:** The same three physical knobs control four different chains depending on the EMU's active MIDI channel. The EMU channel is now both the instrument selector (notes) and the parameter target (CC knobs).

---

## Part 3 — Save Bindings with Snapshot `[draft]`

CC bindings are saved as part of the snapshot state. The bindings from Parts 1 and 2 will be lost if the snapshot is not updated.

### Step 1 — Update the snapshot

Open `http://zynthian.local` → **Snapshots**.

Find `midi-channel-map` (created in [MIDI Channel Routing](project-midi-channel-routing.html)). Click it → **Save** (overwrite with current state including CC bindings).

Alternatively, save a new snapshot named `midi-channel-map-cc` to keep both versions.

### Step 2 — Verify restore

SSH in and reboot:

```bash
reboot
```

Wait ~30 seconds. Reconnect via VNC. Turn knobs 1–3 on the EMU with MIDI channel 2 active.

**Verify:** Filter cutoff and resonance respond immediately — no re-learning needed. The CC badge is visible on the bound knobs.

---

## Going Further

- **Map all 16 knobs** — bind the remaining 13 EMU knobs to envelopes, LFO rate, effects send levels, and other per-chain parameters using the same follow-channel approach
- **CC range limiting** — long-press an already-bound knob → **Set CC Range** → restrict the sweep (e.g. filter cutoff between 400 Hz and 4000 Hz only, so it never closes completely)
- **Block unwanted CCs** — chain_control → MIDI config → CC routing → uncheck any CC numbers the EMU sends but you do not want to reach the engine (e.g. block CC 7 if the EMU auto-sends channel volume and you want the ZynMixer to control levels)
- **ZS3 per section** — save the CC state as a ZS3 subsnapshot for different song sections — one ZS3 with filter open, another with it closed
- **XY control** — bind filter cutoff to X and resonance to Y on the control XY pad for two-dimensional morphing with one touch
