# Maschine MK2 Controller

**Goal:** Connect a Native Instruments Maschine MK2 to Zynthian as a MIDI controller — pads trigger notes, daemon auto-starts on boot.
**Prerequisites:** Zynthian booted and accessible via SSH. A working audio chain already set up (Part 2 requires sound output).
**Access:** SSH

---

## Part 1 — Build, Connect, Verify MIDI `[verified]`

Build the daemon from source, wire it into JACK, and confirm pads send MIDI notes to a Zynthian chain.

The Maschine MK2 has a native USB MIDI class-compliant port, but it sends no data from the pads without Maschine software running. All pad MIDI comes from `MaschineMK2_linux`, a daemon that reads HID data from the device and outputs ALSA MIDI.

### Step 1 — Connect the Maschine MK2

Connect the Maschine MK2 to a USB port on the Raspberry Pi.

**Verify:** USB cable is connected at both ends.

### Step 2 — Find the Maschine USB IDs

```bash
ssh root@zynthian.local
lsusb | grep -i "Native\|Maschine\|17cc"
```

Expected output:
```
Bus 001 Device 003: ID 17cc:1140 Native Instruments Maschine MK2
```

Note the vendor `17cc` and the product ID (e.g. `1140`). You need these in Step 5.

**Verify:** A line with `17cc` appears and you can read the product ID.

### Step 3 — Copy source to the Pi

From the machine where `~/zynth/MaschineMK2_linux` exists:

```bash
ssh root@zynthian.local "mkdir -p ~/zynth"
rsync -av --exclude='target/' ~/zynth/MaschineMK2_linux/ root@zynthian.local:~/zynth/MaschineMK2_linux/
```

**Verify:** Source files present on Pi:
```bash
ls ~/zynth/MaschineMK2_linux/
```

### Step 4 — Install Rust on the Pi

Rust is not installed by default on ZynthianOS:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustc --version
```

Expected: `rustc 1.x.x (...)`

Download and install takes a few minutes on the Pi.

**Verify:** `rustc --version` prints a version number.

### Step 5 — Build the daemon

```bash
cd ~/zynth/MaschineMK2_linux
source "$HOME/.cargo/env"
./build.sh
```

Compiles with link-time optimisation. On a Pi 4 this takes a few minutes — this is normal. Wait for the shell prompt to return.

Expected final lines:
```
Finished `release` profile [optimized] target(s) in ...
move picture to target folder
```

> **Build fix for Rust 1.80+ on ARM64:** If the build fails with `error[E0308]: mismatched types` in `alsa-seq/src/handle.rs`, the ALSA FFI binding uses `*const i8` where newer Rust expects `*const u8`. Fix:
> ```bash
> sed -i 's/as_ptr() as \*const i8/as_ptr() as *const _/' ~/zynth/MaschineMK2_linux/alsa-seq/src/handle.rs
> ```
> Then run `./build.sh` again.

**Verify:**

```bash
ls -lh ~/zynth/MaschineMK2_linux/target/release/maschine
```

File exists (around 600K–700K).

### Step 6 — Add udev rules for stable access and hotplug

Replace `XXXX` with the product ID from Step 2 (e.g. `1140`). Write the file locally, then copy it to the Pi — do not use an inline echo command (quote escaping corrupts the `==` operators):

Create `/tmp/99-maschine.rules` on your local machine:

```
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="17cc", ATTRS{idProduct}=="XXXX", MODE="0664", GROUP="audio", SYMLINK+="maschine"
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="17cc", ATTRS{idProduct}=="XXXX", ACTION=="add", RUN+="/bin/systemctl --no-block restart maschine-mk2.service"
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="17cc", ATTRS{idProduct}=="XXXX", ACTION=="remove", RUN+="/bin/systemctl --no-block stop maschine-mk2.service"
```

Copy and reload:

```bash
scp /tmp/99-maschine.rules root@192.168.2.123:/etc/udev/rules.d/99-maschine.rules
ssh root@192.168.2.123 "udevadm control --reload-rules && udevadm trigger"
```

The three rules together: create a stable `/dev/maschine` symlink, stop the daemon when the USB cable is unplugged, and restart it when plugged back in. The service re-opens the device on each start, so hotplug works cleanly.

**Verify:**

```bash
ssh root@192.168.2.123 "ls -la /dev/maschine"
```

Expected: a symlink pointing to `/dev/hidrawX`.

### Step 7 — Configure a2jmidid to export software MIDI clients

Zynthian uses `a2jmidid` to bridge ALSA MIDI to JACK. By default it only bridges hardware ports. The daemon creates a software ALSA MIDI client, so `a2jmidid` needs the `-e` (export) flag to bridge it.

Add a systemd drop-in override so Zynthian updates do not overwrite this change:

```bash
mkdir -p /etc/systemd/system/a2jmidid.service.d
cat > /etc/systemd/system/a2jmidid.service.d/export-software.conf << 'EOF'
[Service]
ExecStart=
ExecStart=/usr/bin/a2jmidid -e
EOF
systemctl daemon-reload
systemctl restart a2jmidid.service
```

**Verify:**

```bash
pgrep -a a2jmidid
```

Expected: `/usr/bin/a2jmidid -e`

### Step 8 — Create the JACK connect script

The daemon's ALSA MIDI port is bridged to JACK by a2jmidid, but Zynthian's autoconnect does not pick it up automatically (it only connects physical ports). This script finds the port by name and connects it to ZynMidiRouter on every start.

Create the file locally on your Windows machine, then copy it to the Pi. Do not use a heredoc or inline SSH command — shell expansion corrupts the variables.

Create `/tmp/maschine-jack-connect.sh` on your local machine with this exact content:

```bash
#!/bin/bash
for i in $(seq 1 30); do
    PORT=$(jack_lsp 2>/dev/null | grep -m1 'a2j:maschine rs.*Pads MIDI')
    if [ -n "$PORT" ]; then
        jack_connect "$PORT" ZynMidiRouter:dev3_in 2>/dev/null && echo "Connected: $PORT" && exit 0
    fi
    sleep 1
done
echo 'Maschine a2j port not found after 30s'
exit 0
```

Then copy it to the Pi:

```bash
scp /tmp/maschine-jack-connect.sh root@192.168.2.123:/usr/local/bin/maschine-jack-connect.sh
ssh root@192.168.2.123 "chmod +x /usr/local/bin/maschine-jack-connect.sh"
```

**Verify:**

```bash
ssh root@192.168.2.123 "cat -A /usr/local/bin/maschine-jack-connect.sh"
```

Every line must end with `$` only (Unix line endings). If you see `^M$` the file has Windows CRLF — re-copy with a text editor set to LF.

### Step 9 — Run the daemon

```bash
cd ~/zynth/MaschineMK2_linux
./target/release/maschine /dev/maschine any &
```

The `any` argument skips the screen image write. The `&` runs it in the background.

**Verify:** Command returns to a prompt without error. Pad lights on the Maschine may flicker briefly.

### Step 10 — Wire the daemon MIDI port to Zynthian

```bash
/usr/local/bin/maschine-jack-connect.sh
```

Expected output:
```
Connected: a2j:maschine rs [XXX] (capture): Pads MIDI
```

**Verify:**

```bash
aconnect -l | grep maschine
```

Expected: `maschine.rs` listed with a `Pads MIDI` port.

### Step 11 — Play a pad, hear a note

Load a chain with any engine (e.g. ZynAddSubFX with a default preset). Press any pad on the Maschine MK2.

**Verify:** A note plays through audio output when a pad is pressed.

---

## Part 2 — Map Encoders and Buttons `[draft]`

> **Update (2026-06-06):** The daemon now sends standard CC for both encoders (CC 16–23) and transport buttons (CC 1–14, 24–48). Zynthian CC Learn can capture these. This part is ready to be designed and tested. See the Driver Reference below for CC numbers.

This part is ready for implementation. Encoders send CC 16–23 (configurable); transport buttons send CC 1–14 and 24–48 (127 = press, 0 = release).

---

## Part 3 — Run as a Systemd Service `[verified]`

Auto-start the daemon on boot so the Maschine is ready without manual SSH.

### Step 1 — Create the service file

```bash
cat > /etc/systemd/system/maschine-mk2.service << 'EOF'
[Unit]
Description=Maschine MK2 MIDI daemon
After=jack2.service a2jmidid.service
Requires=jack2.service

[Service]
ExecStart=/root/zynth/MaschineMK2_linux/target/release/maschine /dev/maschine any
ExecStartPost=/usr/local/bin/maschine-jack-connect.sh
Restart=always
RestartSec=5
WorkingDirectory=/root/zynth/MaschineMK2_linux

[Install]
WantedBy=multi-user.target
EOF
```

Uses `/dev/maschine` (stable symlink from udev rule) and runs the JACK connect script automatically after the daemon starts.

**Verify:**

```bash
cat /etc/systemd/system/maschine-mk2.service
```

File content matches what was written above.

### Step 2 — Enable and start the service

```bash
systemctl daemon-reload
systemctl enable maschine-mk2.service
systemctl start maschine-mk2.service
```

**Verify:**

```bash
systemctl status maschine-mk2.service --no-pager
```

Expected: `Active: active (running)`

### Step 3 — Reboot and confirm auto-start

```bash
reboot
```

After ~45 seconds, SSH back in:

```bash
ssh root@zynthian.local
systemctl status maschine-mk2.service --no-pager
aconnect -l | grep maschine
jack_lsp -c 2>/dev/null | grep 'maschine.*ZynMidiRouter\|ZynMidiRouter.*dev3'
```

**Verify:** Service is `active (running)`, `maschine.rs` appears in `aconnect -l`, and the JACK connection to `ZynMidiRouter:dev3_in` is present — all without any manual steps.

### Step 4 — Install the Zynthian BPM clock bridge

This service reads Zynthian's current BPM from JACK transport and sends MIDI clock (0xF8 at 24 PPQN) to the Maschine daemon continuously. The Maschine step sequencer locks to this clock when Play is pressed.

Create `/tmp/maschine-clock-bridge.py` on your local machine — get the current version from `~/zynth/MaschineMK2_linux` or copy it from the Pi at `/usr/local/bin/maschine-clock-bridge.py`.

Create `/tmp/maschine-clock-connect.sh` locally:

```bash
#!/bin/bash
for i in $(seq 1 30); do
    CLOCK=$(aconnect -l 2>/dev/null | grep -m1 'RtMidiOut' | grep -oP 'client \K[0-9]+')
    MASCHINE=$(aconnect -l 2>/dev/null | grep -m1 'maschine.rs' | grep -oP 'client \K[0-9]+')
    if [ -n "$CLOCK" ] && [ -n "$MASCHINE" ]; then
        aconnect "$CLOCK:0" "$MASCHINE:1" 2>/dev/null && echo "Clock connected: $CLOCK:0 -> $MASCHINE:1" && exit 0
    fi
    sleep 1
done
echo 'maschine-clock or maschine.rs port not found after 30s'
exit 0
```

Deploy both via scp:

```bash
scp /tmp/maschine-clock-bridge.py root@192.168.2.123:/usr/local/bin/maschine-clock-bridge.py
scp /tmp/maschine-clock-connect.sh root@192.168.2.123:/usr/local/bin/maschine-clock-connect.sh
ssh root@192.168.2.123 "chmod +x /usr/local/bin/maschine-clock-connect.sh"
```

Create the service file on the Pi:

```bash
cat > /etc/systemd/system/maschine-clock.service << 'EOF'
[Unit]
Description=Maschine MK2 JACK transport clock bridge
After=jack2.service maschine-mk2.service
Requires=jack2.service
Wants=maschine-mk2.service

[Service]
ExecStart=/usr/bin/python3 /usr/local/bin/maschine-clock-bridge.py
ExecStartPost=/usr/local/bin/maschine-clock-connect.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl enable maschine-clock.service
systemctl start maschine-clock.service
```

**Verify:**

```bash
systemctl status maschine-clock.service --no-pager
aconnect -l | grep -A4 'maschine.rs'
```

Expected: service `active (running)`, `MIDI Control` shows `Connected From: 128:0, 130:0`.

---

## Part 4 — Web Editor, Config Persistence, and MIDI IN `[draft]`

The daemon includes a browser-based editor for pad LED colors and MIDI configuration, a JSON config file that persists settings across restarts, and an ALSA MIDI input port (`MIDI Control`) that accepts NoteOn messages to drive pad LEDs from an external source.

### Step 1 — Sync and rebuild the daemon

From the machine where the source lives (not the Pi):

```bash
rsync -av --exclude='target/' ~/zynth/MaschineMK2_linux/ root@192.168.2.123:~/zynth/MaschineMK2_linux/
```

Then on the Pi:

```bash
ssh root@192.168.2.123
```

Once on the Pi:

```bash
cd ~/zynth/MaschineMK2_linux
source "$HOME/.cargo/env"
cargo build --release 2>&1 | tail -3
```

Expected last line: `Finished 'release' profile [optimized] target(s) in ...`

**Verify:**
```bash
ls -lh ~/zynth/MaschineMK2_linux/target/release/maschine
```
File exists. Size is typically 600K–1M.

### Step 2 — Restart the daemon service

```bash
systemctl restart maschine-mk2.service
systemctl status maschine-mk2.service --no-pager
```

Expected: `Active: active (running)`

**Verify:** Service is running after restart.

### Step 3 — Open the web editor via SSH tunnel

The daemon's WebSocket server binds to `127.0.0.1:9001` — only accessible locally on the Pi. To reach it from your Windows machine, open an SSH tunnel in a separate terminal:

```bash
ssh -L 9001:127.0.0.1:9001 root@192.168.2.123 -N
```

Keep this terminal open. Then open a browser on your Windows machine and navigate to:

```
http://127.0.0.1:9001
```

Wait up to 5 seconds for the WebSocket connection to establish.

**Verify:** The web editor loads and shows a pad grid or connection status.

[low] Exact web UI layout and control labels need Pi verification.

### Step 4 — Change a pad LED color

In the web editor, click any pad in the grid. Select a color.

**Verify:** The corresponding pad on the Maschine MK2 hardware lights in the selected color within 1–2 seconds.

[low] Requires working SSH tunnel from Step 3.

### Step 5 — Verify config persistence

In the web editor config panel, change the CC number for Encoder 1 from 16 to a different value (e.g. 20).

Then restart the daemon:

```bash
systemctl restart maschine-mk2.service
```

Check the saved config:

```bash
cat ~/zynth/MaschineMK2_linux/maschine.json
```

Expected: `encoder_ccs` shows `20` in the first position.

**Verify:** Encoder CC change survives a daemon restart.

[low] Requires working SSH tunnel from Step 3.

### Step 6 — Drive pad LEDs from MIDI IN

Connect any MIDI source to the `MIDI Control` ALSA port:

```bash
aconnect -l | grep -A3 maschine
```

Note the client number (e.g. `28`) and port `1` (MIDI Control). Connect a source:

```bash
aconnect <source-client>:<port> 28:1
```

Replace `28` with the actual maschine.rs client number. For a quick test, connect the Xboard:

```bash
aconnect <xboard-client>:0 28:1
```

Press keys in the range MIDI notes 0–15 on the connected source (on a standard keyboard, these are very low notes — C-1 to D#0).

**Verify:** Pad LEDs on the Maschine light in response to incoming NoteOn messages. Velocity controls brightness.

[low] Exact `aconnect` client numbers and best test procedure need Pi verification.

### Step 7 — Check the display

Look at the Maschine MK2's 128×64 OLED display while the daemon is running.

**Verify:** The display shows note names or encoder CC values.

[low] Display content and layout need Pi verification.

---

**Verify (Part 4 complete):** Web editor loads and connects, pad LED changes on color set, `maschine.json` shows updated CC after restart, NoteOn to MIDI Control port lights corresponding pads.

---

## Driver Reference

The `MaschineMK2_linux` daemon reads HID data from `/dev/maschine` and outputs ALSA MIDI on the `Pads MIDI` port (Ch1). All MIDI goes out on channel 1.

### Pads (16)

- **Note On/Off** — velocity from pressure, exponential curve (power 0.4)
- **Polyphonic aftertouch** — compiled in, disabled in current build
- **Note layout:** pad positions map to offsets `[12,13,14,15, 8,9,10,11, 4,5,6,7, 0,1,2,3]` added to the current note base
- **Pad lights** — RGB color + brightness, refreshed at ~60 fps

### Group Buttons A–H

Set the MIDI note base for all pads:

| Button | Note base |
|--------|-----------|
| A | 24 (C1) |
| B | 36 (C2) |
| C | 48 (C3) |
| D | 60 (C4, middle C) |
| E | 72 (C5) |
| F | 84 (C6) |
| G | 96 (C7) |
| H | 108 (C8) |

### 8 Encoders

Send standard **CC** on Ch1. Default CC numbers: 16–23 (Encoder 1 = CC 16, Encoder 8 = CC 23). Values 0–127. CC Learn can capture these.

CC numbers are configurable per-encoder via `maschine.json` or the web editor (see Part 4).

### Transport and Function Buttons

Send standard **CC** on Ch1. Value 127 = button pressed, 0 = button released. CC Learn can capture these.

| Button | CC number |
|--------|-----------|
| Play | 1 |
| Stop (Erase button) | 2 |
| Rec | 3 |
| Grid | 4 |
| Step Left | 5 |
| Step Right | 6 |
| Restart | 7 |
| Browse | 8 |
| Sampling | 9 |
| Note Repeat | 10 |
| Control | 11 |
| Nav | 12 |
| Nav Left | 13 |
| Nav Right | 14 |
| Main | 24 |
| Scene | 25 |
| Pattern | 26 |
| Pad Mode | 27 |
| View | 28 |
| Duplicate | 29 |
| Select | 30 |
| Solo | 31 |
| Step | 32 |
| Mute | 33 |
| Navigate | 34 |
| Tempo | 35 |
| Enter | 36 |
| Auto | 37 |
| All | 38 |
| F1–F8 | 39–46 |
| Page Right | 47 |
| Page Left | 48 |

### Shift Button

Hold Shift to activate modifier state. Shift + Pad Mode enters pad mode 1. Shift + encoder B6 sets the step sequencer speed.

### Step Sequencer (Pad Mode 2)

Activated by Shift + Pad Mode a second time. Pads toggle steps on/off instead of playing notes. Play starts playback; Erase stops it. Speed set by Shift + encoder B6 [low].

**Group buttons A–H** switch between 8 independent 16-step pages in sequencer mode (not note base — that is pad mode behaviour).

**Per-step editing:** tap a step to select it (orange LED), then Encoder 1 = velocity (0–127), Encoder 2 = note offset (0–127).

**Euclidean fill:** Shift + Group A–H fills the current page with 1–8 evenly distributed hits.

**MIDI clock sync:** external MIDI clock received on the `MIDI Control` input port locks the step rate (6 ticks = one 16th-note step). Fallback to internal BPM after 500 ms of clock silence.

### OSC Interface

The daemon listens on `127.0.0.1:42434` and sends to `127.0.0.1:42435`.

**Incoming (control the hardware LEDs):**

| Path | Arguments | Effect |
|------|-----------|--------|
| `/maschine/button/<name>` | `brightness` | Set button LED white at brightness |
| `/maschine/button/<name>` | `color_int brightness` | Set button LED color + brightness |
| `/maschine/pad` | `pad_idx color_int brightness` | Set pad LED |
| `/maschine/midi_note_base` | `base` | Set MIDI note base remotely |

**Outgoing (every hardware event):**

- Each button event: `/<button_name> <value>`
- Each encoder step: encoder index + value

---

## Going Further

- Use Group buttons A–H to switch octaves and cover different note ranges across the 16 pads
- Map Play and Stop buttons to Zynthian transport controls via CC Learn
- Assign all 8 encoders to different synth parameters for a custom performance layout
- Save a named snapshot with all CC bindings — Zynthian restores them automatically on load
