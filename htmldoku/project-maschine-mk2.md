# Maschine MK2 Controller

**Goal:** Connect a Native Instruments Maschine MK2 to Zynthian as a fully functional MIDI controller — pads trigger notes, buttons and encoders map to synth parameters, daemon auto-starts on boot.
**Prerequisites:** Zynthian booted and accessible via SSH. A working audio chain already set up (Part 2 requires sound output). The `MaschineMK2_linux` source, Rust toolchain, and compiled binary are already present on this Pi — Steps 3 and 4 can be skipped.
**Access:** SSH

---

## Part 1 — Build, Connect, Verify MIDI `[draft]`

Build the daemon from source, set permissions, run it manually, and confirm pads send MIDI notes to a Zynthian chain.

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

### Step 3 — Install Rust on the Pi

Rust is not installed by default on ZynthianOS:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustc --version
```

Expected: `rustc 1.x.x (...)`

Download and install takes a few minutes on the Pi.

**Verify:** `rustc --version` prints a version number.

### Step 4 — Build the daemon

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

### Step 5 — Add a udev rule for stable device access

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

### Step 6 — Run the daemon

```bash
cd ~/zynth/MaschineMK2_linux
./target/release/maschine /dev/maschine any &
```

The `any` argument skips the screen image write. The `&` runs it in the background.

**Verify:** Command returns to a prompt without error. Pad lights on the Maschine may flicker briefly.

### Step 7 — Confirm MIDI port is visible

```bash
aconnect -l
```

Expected:
```
client 32: 'maschine.rs' [type=kernel]
    0 'maschine.rs port 0'
```

**Verify:** `maschine.rs` appears in the output.

### Step 8 — Enable the port in webconf

Open `http://zynthian.local` → **Interface → MIDI Options**. Click **MIDI Devices**. Find `maschine.rs` and enable it.

**Verify:** Port is toggled on.

### Step 9 — Play a pad, hear a note

In VNC, load a chain with any engine (e.g. ZynAddSubFX with a default preset). Press any pad on the Maschine MK2.

**Verify:** A note plays through audio output when a pad is pressed. Group buttons A–H shift the note base up or down.

---

## Part 2 — Map Buttons and Encoders via CC Learn `[draft]`

Bind Maschine buttons and encoders to synth parameters using Zynthian's CC Learn system.

### Step 1 — Open a synth chain in VNC

In the Zynthian VNC UI, tap a chain to open its control screen. Parameter knobs (e.g. Cutoff, Resonance, Volume) should be visible.

**Verify:** Chain control screen with parameter knobs is visible.

### Step 2 — Enter CC Learn on a parameter

Long-press a parameter knob in the VNC UI (hold ~600ms). The knob highlights orange — CC Learn is now active for that parameter.

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

## Part 3 — Run as a Systemd Service `[draft]`

Auto-start the daemon on boot so the Maschine is ready without manual SSH.

### Step 1 — Create the service file

```bash
cat > /etc/systemd/system/maschine-mk2.service << 'EOF'
[Unit]
Description=Maschine MK2 MIDI daemon
After=jack2.service
Requires=jack2.service

[Service]
ExecStart=/root/zynth/MaschineMK2_linux/target/release/maschine /dev/maschine any
Restart=on-failure
RestartSec=3
WorkingDirectory=/root/zynth/MaschineMK2_linux

[Install]
WantedBy=multi-user.target
EOF
```

Uses `/dev/maschine` — the stable symlink from the udev rule. No hardcoded hidraw number needed.

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
```

**Verify:** Service is `active (running)` and `maschine.rs` appears in `aconnect -l` without any manual steps.

---

## Going Further

- Use Group buttons A–H to switch octaves and cover different note ranges across the 16 pads
- Map Play and Stop buttons to Zynthian transport controls via CC Learn
- Assign all 8 encoders to different synth parameters for a custom performance layout
- Save a named snapshot with all CC bindings — Zynthian restores them automatically on load
