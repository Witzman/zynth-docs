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

### Step 6 — Add a udev rule for stable device access

Replace `XXXX` with the product ID from Step 2 (e.g. `1140`):

```bash
echo 'SUBSYSTEM=="hidraw", ATTRS{idVendor}=="17cc", ATTRS{idProduct}=="XXXX", MODE="0664", GROUP="audio", SYMLINK+="maschine"' > /etc/udev/rules.d/99-maschine.rules
udevadm control --reload-rules
udevadm trigger
```

This creates a stable path `/dev/maschine` regardless of USB port or boot order.

**Verify:**

```bash
ls -la /dev/maschine
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

```bash
cat > /usr/local/bin/maschine-jack-connect.sh << 'EOF'
#!/bin/bash
for i in $(seq 1 30); do
    PORT=$(jack_lsp 2>/dev/null | grep -m1 'a2j:maschine rs.*Pads MIDI')
    if [ -n "$PORT" ]; then
        jack_connect "$PORT" ZynMidiRouter:dev3_in 2>/dev/null && echo "Connected: $PORT" && exit 0
    fi
    sleep 1
done
echo 'Maschine a2j port not found after 30s'
exit 1
EOF
chmod +x /usr/local/bin/maschine-jack-connect.sh
```

**Verify:**

```bash
ls -lh /usr/local/bin/maschine-jack-connect.sh
```

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

## Part 2 — Map Buttons and Encoders via CC Learn `[draft]`

Bind Maschine buttons and encoders to synth parameters using Zynthian's CC Learn system.

### Step 1 — Open a synth chain

On the touchscreen, tap a chain to open its control screen. Parameter knobs (e.g. Cutoff, Resonance, Volume) should be visible.

**Verify:** Chain control screen with parameter knobs is visible.

### Step 2 — Enter CC Learn on a parameter

Long-press a parameter knob on the touchscreen (~600ms). The knob highlights orange — CC Learn is now active for that parameter.

**Verify:** Knob turns orange.

### Step 3 — Move a Maschine button or encoder

While the knob is orange, press a Maschine button or turn a Maschine encoder. Zynthian captures the CC number and binds it.

Note on encoders: they send absolute CC values (0–127), not relative increments. They work best for parameters where jumping to a value is acceptable. Buttons send CC 127 on press — useful for on/off toggles.

**Verify:** Knob returns to normal colour. Moving the same Maschine control now affects the parameter.

### Step 4 — Repeat for other parameters

Repeat Steps 2–3 for each parameter you want to control.

**Verify:** All desired CC bindings respond correctly.

### Step 5 — Save a snapshot

CC bindings are stored in snapshots. Save now so they survive reboots:

In webconf, go to **Library → Snapshots**. Type a name in the **Name:** field and click the checkmark button to save.

**Verify:** Snapshot appears in the list.

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
Restart=on-failure
RestartSec=3
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

---

## Going Further

- Use Group buttons A–H to switch octaves and cover different note ranges across the 16 pads
- Map Play and Stop buttons to Zynthian transport controls via CC Learn
- Assign all 8 encoders to different synth parameters for a custom performance layout
- Save a named snapshot with all CC bindings — Zynthian restores them automatically on load
