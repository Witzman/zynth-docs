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

---

## Session 2026-08-08 — geometry measured, not guessed

A live test rig replaced the rebuild-per-guess loop. Three new OSC paths on the
daemon (commit `e20e560` onward):

| Path | Args | Does |
|---|---|---|
| `/maschine/display/test` | pattern:int | draw a built-in calibration pattern |
| `/maschine/display/opts` | col, reverse, bands | change framing without a rebuild |
| `/maschine/display/calib` | on:int | interactive line calibration on encoders 1-4 |
| `/maschine/display/clear` | — | blank both screens |

Helper scripts on the Pi: `/root/disp.py`, `/root/disp_sweep.py`,
`/root/disp_bands.py`, `/root/disp_tiles.py`.

### Confirmed by experiment

- **Each screen is 512 px wide, not 128.** One report is a 128x32 tile. Painting
  at column offsets 0, 8, 16, 24 took the lit area from a quarter to a half to
  the whole screen with no seam or gap. `WIDTH` was corrected to 512 and
  `send_display_bits` now cuts 8 tiles (4 columns x 2 row bands) out of a full
  framebuffer. This is what "readable but too big" was: text filled a quarter of
  the panel and looked magnified.
- **Header byte 1 is a column offset in 16-pixel units.** Eight offsets
  (0, 4 ... 28) tile the width; 7 steps to cross.
- **Header byte 3 is a row offset.** One band = half the height, two = full.
- **Row 0 across the full width renders solid, edge to edge, on both screens.**
  Horizontal geometry is correct.
- **A single text row at y=0 renders correctly and legibly.** Photographed:
  four labels `A KICK / B SNARE / C HAT / D CLAP` across the width, well
  proportioned, aligned under the four buttons. **This is usable now.**

### Interactive calibration result

Lines dialled to the last visible position gave **x 0..446, y 0..47**.

### Still unresolved — the vertical mapping

Contradictory observations that no single linear scaling explains:

- A 1-row horizontal line renders **2 px tall**.
- A 1-column vertical line renders **dotted, every other row**.
- The row ruler (lines every 8 rows) shows only **4 lines**, evenly spaced, the
  5th just past the bottom edge — implying ~32 usable rows.
- But the band test showed one band = half height, two = full — implying 64.
- Content below y~8 is **partially dropped**: 8px glyphs at y=12 render as
  fragments, and 8px-tall bar outlines collapse to thin lines. The lower half of
  the panel stays unused.

Working theory: framebuffer rows do not map 1:1 to physical rows past the first
band, possibly interleaved, so consecutive rows land 2 apart and tall elements
smear and overlap. Not yet proven.

### Next steps

1. Draw single rows one at a time (y=0, then 1, 2, 3, 8, 9) and record where each
   lands. That maps the row order directly and settles interleaving.
2. Until then, **use only the top text row** — it is verified working. Labels
   under the F buttons are deliverable; multi-row layouts and bars are not.
3. `usbmon` capture remains the fallback for the row mapping.

### Note

`write_display()` is still not wired to anything, and normal rendering stays off
in `ev_loop` (it issued ~180 writes/s of 521-byte reports). Calibration redraws
are rate-limited to the existing 100ms display timer. Redrawing per input report
starved the input reader and tripped the hidraw watchdog.

Two bugs worth remembering: `encoder_step` receives the encoder's **absolute
counter byte**, not a delta (`mikro.rs:415` passes `byte as i32`) - treating it
as a delta pinned every calibration line instantly. And `send_encoder_cc`
divides that raw value by 4, so 4 counts is one pixel.

---

## 2026-08-09 (later) — GEOMETRY SOLVED AND HARDWARE-VERIFIED. Read only this section.

**Everything below this section about geometry is superseded.** The panel is **255x64, 1bpp row-major, MSB = leftmost pixel**, 32 bytes per row (the 256th column is transferred and discarded). One screen is **8 HID reports**, each a full-width band of 8 rows:

```
header = [0xE0|screen, 0x00, 0x00, chunk*8, 0x00, 0x20, 0x00, 0x08, 0x00]
payload = 256 bytes = framebuffer[chunk*256 .. +256]     chunk = 0..7
report length = 9 + 256
```

- **Header byte 5 = bytes per row (0x20 = 32 → 255 px), byte 7 = rows per report (0x08).** This driver had those two **swapped**, so the panel was told to expect a 64x32 region while being fed 512 bytes laid out 128 px wide. That single swap is the whole reason the screens garbled, through every earlier theory.
- **Byte 1 is an x offset in bytes, byte 3 a y offset in rows.** Reading byte 1 as a 16-pixel unit is where "the panel is 512 wide" came from: the offsets 0/8/16/24 span 0..256, not 0..512.
- No tiling, no column strips, no row bands, no page/column conversion, no bit reversal, no x scaling. The framebuffer slices straight into the 8 reports.

Source of truth: **cabl** (`shaduzlabs/cabl`, `src/devices/ni/MaschineMK2.cpp`) — known-working, MIT. Its `sendFrame` is the transfer above verbatim, and its `GDisplayMaschineMK2::setPixel` is `data[widthBytes*y + (x>>3)] |= 0x80 >> (x&7)`, identical to `display.rs`. Three months of guessing lost to not reading it.

**Verified on hardware 2026-08-09:**

- Border rect on all four edges; lines at x=248, 251, 254 all render with **254 in the last physical column**, x=255 shows nothing → width is 255.
- Text at y=2 (5x8), y=24 (double height) and y=48 (5x8) all render complete and correctly placed, full width, no wrap.
- The full rig layout renders clean: four group tabs, dotted rule, four encoder columns with name + double-height value + indicator bar. Nothing cut off or cramped.

Code: `MaschineMK2_linux` `bbf2a62`. `clear_screen` now goes through `send_display_bits` with a blank buffer instead of its own hand-rolled sweep. `display::CHUNKS/CHUNK_ROWS/CHUNK_BYTES/HDR_ROW_BYTES/HDR_ROWS` carry the numbers; a unit test asserts the chunks tile the framebuffer exactly.

Layout mock on the Pi: `/root/mock2.py` (255-wide, four 64-px columns, bars at y 52-61). Probe tool: `/root/disp2.py`.

---

## Superseded history removed 2026-08-10

Everything below this line used to hold the wrong geometry models — 128x128, 512x64,
128x32 tiles, two row bands per tile, "rows are 2 px tall", "transfer rows discarded".
They were marked superseded and a subagent read them anyway and planned an hour of work
to re-verify a solved problem. The full history is in git if it is ever wanted:
`git log --follow MD/display-investigation.md`.

The section above is the only correct description.
