# Maschine MK2 Display Investigation

## Goal
Get the two 128×? OLED displays working. Each display: report ID 0xE0 (left) / 0xE1 (right).
HID report buffer: `[0u8; 1 + 8 + 512]` = 521 bytes (1 report ID + 8 header + 512 data).

## Hardware
- Two monochrome OLED displays on Maschine MK2 (full size, not Mikro)
- Physical appearance: narrow horizontal strips (likely 128×32 each)
- Connected via HID to USB — separate from pad/button HID path
- Daemon on Pi: `/root/zynth/MaschineMK2_linux/target/release/maschine`

## Source Files
- `src/devices/mk2/mikro.rs` — `send_display_bits()`, `write_display()`, `clear_screen()`
- `src/display.rs` — `WIDTH=128`, `HEIGHT` (see attempts), `STRIDE=WIDTH/8=16`
- `src/font.rs` — `FONT5X8`: column-major, 5 cols × 8 rows, bit 0 = top row, bit 7 = bottom row

## HID Report Header (bytes 0-8)
```
[0]  = report_id (0xE0 left, 0xE1 right)
[1]  = x_start (column offset — possibly in pixels)
[2]  = 0
[3]  = y_start (row offset — possibly in pixels or pages)
[4]  = 0
[5]  = 0x08 (fixed — unknown meaning, possibly width descriptor)
[6]  = 0
[7]  = 0x20 = 32 (fixed — possibly height descriptor)
[8]  = 0
[9..521] = 512 bytes pixel data
```

## clear_screen (working — display goes blank at startup)
Sends 64 reports per display (8 col values × 8 page values), all zero data.
- buf[1] (col) = k*4 for k in 0..8 (values 0,4,8,...,28)
- buf[3] (page) = t*4 for t in 0..8 (values 0,4,8,...,28)
- Off-by-one: page updates at end of each k-cycle, so first 7 writes per cycle use previous t's page
- All-zero data regardless of position → display cleared to black

## Pixel Format (display.rs)
Row-major 1bpp: `bits[y * STRIDE + x/8] |= 0x80 >> (x % 8)`
- Bit 7 (MSB) of byte = leftmost pixel of that 8-pixel group
- Row 0 = bytes 0..15, Row 1 = bytes 16..31, etc.

## Attempts Log

### Attempt 1 — raw row-major, 2 reports, HEIGHT=64 (BEST SO FAR)
```rust
// Report 1: rows 0-31 (bytes 0-511)
buf[3] = 0;  buf[9..521] = bits[0..512];
// Report 2: rows 32-63 (bytes 512-1023)
buf[3] = 32; buf[9..521] = bits[512..1024];
```
**Result:** "Readable but too big" — content visible, characters recognizable.
Text drawn at y=0,10,20. All within rows 0-31 so bytes 512-1023 are all zeros.

### Attempt 2 — page-column conversion, 2 reports, HEIGHT=64
Converted row-major → page-column format before sending.
**Result:** "Pixeltrash in upper 1/8 of display" — worse than attempt 1.
Conclusion: hardware uses row-major format directly. No conversion needed.

### Attempt 3 — raw row-major, 1 report, HEIGHT=32
```rust
// Only one 512-byte report
buf[3] = 0;  buf[9..521] = bits[0..512];
```
HEIGHT changed to 32, buffer shrinks to 512 bytes.
**Result:** Different garbling — "KX3 KX4" on left display, "KX7 KX8" on right.
- 'K' visible correctly, digit after K garbled (two chars overlaid appearance)
- Second row "00 00" but bottom pixels of '0' cut off (looks like inverse 'A')
- Third line (BASE:...) not mentioned — may be missing

### Attempt 4 — restore attempt 1 (HEIGHT=64, 2 reports)
**Result:** "Same as before" = back to "readable but too big" state. ✓

## Current State (as of 2026-06-06)
- Best state: HEIGHT=64, 2 reports, raw row-major → "readable but too big"
- Service: `maschine-mk2.service` active, display showing partial content
- Local source at commit state after attempt 4 (HEIGHT=64, 2 reports)

## What "Readable But Too Big" Means
Characters are recognizable but font/layout appears oversized for the physical display.
Possible causes:
1. Display is actually 128×32 but we're rendering for 128×64 (content squashed or partial)
2. Column offset (64px?) shifts content so only right half of string visible
3. Byte3=32 in second report corrupts display controller state

## Key Unknowns

### Why does HEIGHT=32 single-report produce different garbling than HEIGHT=64 2-report?
Both should send identical bytes 0-511. The second report in attempt 1 sends all-zero bytes (rows 32-63 empty). Removing it shouldn't change what rows 0-31 show. Unless the second report triggers a display controller commit/refresh.

### Column start offset
"KX3 KX4" in attempt 3 — in the string " K1    K2    K3    K4", K3 appears at pixel ~78 and K4 at ~114. If only K3/K4 are visible, a 64-pixel column offset would shift content so col 64-127 wraps to left side. To test: set buf[1]=64 and see if K1/K2 become visible.

### Display height
Physical strips appear narrow. 128×32 likely but unconfirmed. Could be 128×48 or even 128×64 with small pixels.

### Bit order
display.rs uses MSB=leftmost. If hardware expects LSB=leftmost, all content mirrored horizontally. Test: reverse bits in each byte before sending.

## Next Debug Steps (priority order)

1. **Column offset test** — try buf[1]=64 with current (HEIGHT=64, 2 reports). Does K1/K2 move to left side?
2. **Bit reversal test** — reverse bits in each byte (`byte.reverse_bits()`). Does content de-mirror?
3. **Single report, HEIGHT=64** — send only the first report (omit byte3=32). Confirm if second report is required or if its absence produces same result as attempt 3.
4. **USB capture** — `usbmon` on Pi to capture actual HID traffic and compare working `clear_screen` with `send_display_bits`.
   ```bash
   modprobe usbmon
   # find bus: lsusb | grep -i native   
   tcpdump -i usbmon1 -w /tmp/maschine-usb.pcap
   ```
5. **Reference implementation** — search for Maschine MK2 display HID protocol in maschine-rs, ni-controllers, or similar projects.

## Related Code Locations
```
src/devices/mk2/mikro.rs:431  — send_display_bits()
src/devices/mk2/mikro.rs:877  — write_display()  (approximately, shifts with edits)
src/display.rs:1              — WIDTH, HEIGHT, STRIDE constants
src/display.rs:32             — draw_text()
src/font.rs:1                 — FONT5X8 glyph data
```
