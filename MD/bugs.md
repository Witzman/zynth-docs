# Bugs

## Open

### BLE MIDI broken on kernel 6.12 (zynbluez 5.76)

**Symptom:** `bluetoothctl connect` fails with `le-connection-abort-by-local`. BlueZ debug shows `att_connect_cb() connect to ...: Function not implemented (38)`.

**Root cause:** zynbluez 5.76 was compiled in May 2024 against kernel 6.6 headers. The L2CAP ATT socket interface changed in kernel 6.10+. The Pi now runs 6.12.47+rpt-rpi-v8.

**Workaround:** Connect SMC-PAD via USB-C instead of BLE. JACK `-X raw` handles USB MIDI natively — device appears as `system:midi_capture_N` automatically.

**Fix:** Rebuild zynbluez 5.76 from source against kernel 6.12 headers, or wait for Zynthian package update.

**Affects:** SMC-PAD BLE connection (and any other BLE MIDI device).

### JACK fails when U46DJ not connected

**Symptom:** `systemctl status jack2` shows `exit-code` / `EXCEPTION` on boot if U46DJ is not plugged in or powered on. Zynthian starts but has no audio. `journalctl -u jack2` shows `Cannot open device hw:U46DJ`.

**Root cause:** jack2.service ExecStart is configured for `hw:U46DJ` (done during U46DJ audio setup). JACK cannot open a device that is absent.

**Workaround:** Power on and connect U46DJ before booting, or before running `systemctl start jack2`.

**Affects:** All tutorials that require audio output (Drone Synth, MIDI Channel Routing, Audio FX, Performance Rig).

---

### E-MU Xboard25 ALSA client name unstable across reboots

**Symptom:** After some reboots, the Xboard appears in `aconnect -l` as `USB Device 0x41e:0x3f00` instead of `E-MU Xboard25`. The device still works — only the displayed name differs.

**Root cause:** USB enumeration order or kernel driver path determines which name string is used. Not consistent.

**Workaround:** Use `lsusb | grep 041e` to confirm the device is connected, regardless of the name shown in `aconnect -l`.

**Affects:** Any tutorial step that asks the user to identify the Xboard by name in `aconnect -l`.

---

### SINCO (SMC-PAD) card number changes across reboots

**Symptom:** The card number in `aconnect -l` (e.g. `card=3`) differs between sessions. `amidi -p hw:X,0,1` requires the current card number each time.

**Root cause:** USB device enumeration order is not fixed — card numbers are assigned dynamically at boot.

**Workaround:** Always run `aconnect -l` at the start of a session to find the current card number before using `amidi`.

**Affects:** SMC-PAD Launcher Control tutorial Part 1 Step 3.

---

### Maschine daemon crash loop — broken jack-connect script

**Symptom:** `maschine-mk2.service` in `activating (auto-restart)` loop, 1200+ restarts. Journal shows `syntax error near unexpected token '2'` on line 4 of `/usr/local/bin/maschine-jack-connect.sh`.

**Root cause:** `/usr/local/bin/maschine-jack-connect.sh` was corrupted — `$(seq 1 30)` expanded to literal newlines in the for loop, and the `PORT=$(jack_lsp ...)` grep line was missing entirely. `ExecStartPost` failed with `status=2/INVALIDARGUMENT`, causing systemd to kill the main daemon process.

**Fix (applied 2026-06-04):** Rewrote the script via `python3` on the Pi to avoid heredoc expansion issues. Script now correct. Service confirmed `active (running)`, connection log: `Connected: a2j:maschine rs [129] (capture): Pads MIDI`.

**Affects:** Maschine MK2 daemon — pads send no MIDI to Zynthian while crashed.

---

### TOGGLE_SEQ never fires — three compounding bugs (2026-06-04, partially resolved)

**Symptom:** Pressing SMC-PAD pads plays notes on active chain but never triggers TOGGLE_SEQ launcher slot toggle, despite 16 master key actions configured.

**Root causes found (in order of discovery):**

1. **Wrong master channel in config** — `ZYNTHIAN_MIDI_MASTER_CHANNEL` was 7 but SMC-PAD sends on channel 6 (1-indexed), not 7. Status byte `0x95` = Note-On ch6 (0-indexed 5). Config corrected to 6. Python converts 1-indexed to 0-indexed so master_chan in C = 5.

2. **`MASTER_NOTE_CUIA` parser requires literal `\n` separators** — The env var parser in `zynthian_gui_config.py` does `split('\\n')` (2-char backslash-n). Writing actual newlines to `default.sh` caused silently empty note_cuia dict. Rewrote the value with `\n` (literal backslash-n) separators using Python lambda in re.sub to avoid interpreter escaping.

3. **SINCO Private port doubles every event** — SINCO SMC-PAD Private (ALSA card 4 port 0 = `system:midi_capture_3` = SINCO IN 1) mirrors ALL pad MIDI from SINCO Master (port 1 = `system:midi_capture_4` = SINCO IN 2). Both connect to ZynMidiRouter. TOGGLE_SEQ fires twice per pad press → double-toggle → no net change. Debounce added to `zynthian_state_manager.py` on Pi: 50ms window per note (lines 836–840).

**Partial fix applied:**
- `ZYNTHIAN_MIDI_MASTER_CHANNEL=6` in `/zynthian/config/midi-profiles/default.sh`
- All 16 `TOGGLE_SEQ` mappings written with `\n` separators
- 50ms debounce in state_manager.py (Pi only, not committed to zynthian-ui git)

**Still not working after debounce fix.** Possible remaining causes:
- Launcher has no patterns loaded → `togglePlayState` succeeds silently
- `cuia_toggle_seq` only uses `params[0]` (flat sequence index) — format may be wrong
- `_master_cuia_last` debounce dict init may not be applied (check state_manager.__init__)

**Affects:** SMC-PAD Launcher Control tutorial Part 3 — all TOGGLE_SEQ pad mappings.

---

### Maschine daemon — display floods USB, pads stop working (2026-06-06)

**Symptom:** After driver update (13 commits), pads lit but sent no MIDI. Display showed random pixels in upper portion of both screens. Daemon stuck in kernel D state.

**Root cause:** `send_display_bits()` sends one byte per 521-byte HID report. `write_display()` calls it for a 1024-byte buffer per display × 2 displays = 2048 USB writes per 100ms call. USB write queue saturates → daemon blocks in uninterruptible D state → `readable()` never called → no pad events.

**Fix (applied 2026-06-06):** Disabled `write_display()` with early return in `mikro.rs`. Display shows blank. `send_display_bits` needs rewrite to pack all display bytes into proper 512-byte bulk HID chunks before display can be re-enabled.

**Commit:** `ffc8f2b` in MaschineMK2_linux.

**Affects:** All driver versions that call `write_display()` at runtime. Pre-update version never called it, so display worked (blank = cleared once at startup).

---

### Maschine daemon — connect script corrupted again (2026-06-06, third incident)

**Symptom:** `maschine-mk2.service` crash-looping (114+ restarts). Journal: `syntax error near unexpected token '2'` line 4. Pads dark, no MIDI.

**Root cause:** `/usr/local/bin/maschine-jack-connect.sh` was corrupted — `$(seq 1 30)` expanded to individual numbers each on its own line, `PORT=$(jack_lsp ...)` reduced to `PORT=`. Script could not parse. Happened after previous DBUS fix session; exact cause unknown (likely SSH heredoc from previous session without quoted EOF).

**Fix (applied 2026-06-06):** Wrote corrected script locally, deployed via `scp`. DBUS line not re-added (jack_lsp connects without it in current systemd env). `exit 1` corrected to `exit 0` so timeout is non-fatal. Service confirmed running: `Connected: a2j:maschine rs [129] (capture): Pads MIDI`.

**Prevention:** Always deploy this script via `scp` from a local file. Never use heredoc or SSH inline commands — shell expansion corrupts it.

**Affects:** Maschine MK2 daemon — all MIDI silently broken.

---

### Maschine daemon — JACK connection missing after restart (2026-06-06)

**Symptom:** Pads light up and ALSA port `129:0 → 128:0` shows connected, but pads produce no sound. ZynMidiRouter `dev3_in` has no connections.

**Root cause:** `maschine-jack-connect.sh` post-start script ran `jack_lsp` without `DBUS_SESSION_BUS_ADDRESS` set — JACK uses SHM IPC and requires the correct DBUS env to connect from systemd context. Script always timed out, was rewritten to exit 0, leaving `a2j:maschine rs [129] (capture): Pads MIDI` unconnected to `ZynMidiRouter:dev3_in`. ALSA `128:0` connection is not the active MIDI path — Zynthian uses JACK `devN_in` ports.

**Fix (applied 2026-06-06):** Rewrote `/usr/local/bin/maschine-jack-connect.sh` to set `DBUS_SESSION_BUS_ADDRESS=unix:path=/run/dbus/system_bus_socket` before calling `jack_lsp`/`jack_connect`. Script exits 0 on timeout (non-fatal) so daemon stays alive. Manual `jack_connect` applied immediately.

**Affects:** Every daemon restart — MIDI silently broken until manual `jack_connect`.

---

### Maschine web editor — pad LED color targets wrong physical pad (2026-06-06, fixed)

**Symptom:** Setting color for pad N in the web editor lights a different physical pad on the hardware.

**Root cause:** `set_pad_light(pad, ...)` in `mikro.rs` used `pad` directly as the LED HID report index. The LED output report uses display order (top-left first, matching `PAD_DISPLAY_ORDER`), but the input report uses bottom-up row-major order. Index 0 in the LED report = top-left physical pad, but input index 0 = bottom-left physical pad.

**Fix (applied 2026-06-06):** Added `PAD_LED_MAP = [12,13,14,15,8,9,10,11,4,5,6,7,0,1,2,3]` remapping in `set_pad_light`. This is identical to `PAD_DISPLAY_ORDER`, which is its own inverse (an involution), so it correctly maps input index → LED buffer position in both directions.

**Commit:** `1fb62eb` in MaschineMK2_linux.

**Affects:** Web editor pad color commands and any `set_pad_light` call using input-order pad indices.

---

## Closed

### Touchscreen double-click — X11 claiming device as pointer (2026-06-06, fixed)

**Symptom:** Single tap on touchscreen registered as double-click in Zynthian UI.

**Root cause:** `/etc/X11/xorg.conf.d/99-elecrow-touch.conf` had `Option "Ignore" "false"`. X11 registered `wch.cn USB2IIC_CTP_CONTROL` as a slave pointer and generated X pointer events. `multitouch.py` reads the same device via `/dev/input/event0` directly (evdev). Every physical tap produced one X11 pointer event + one multitouch.py event → two actions.

`multitouch.py` has xinput-disable logic in `open_device()` to prevent this, but it was not firing reliably — `open_device()` silently swallows exceptions and the disable was not confirmed working at startup.

**Fix (applied 2026-06-06):** Set `Option "Ignore" "true"` in `/etc/X11/xorg.conf.d/99-elecrow-touch.conf`. X11 no longer claims the device as a pointer. `multitouch.py` still reads `/dev/input/event0` directly — touch works normally, no double events. Takes effect after X11 restart (reboot applied).

**Affects:** All UI interaction — every tap triggered two actions.

