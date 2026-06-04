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

## Closed

