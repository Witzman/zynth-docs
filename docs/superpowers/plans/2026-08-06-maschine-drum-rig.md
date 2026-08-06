# Maschine MK2 Drum Rig Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the Maschine MK2 into an 8-track × 16-step euclidean drum sequencer for live techno, driven by a new Zynthian ctrldev driver with zynseq as the sequencer.

**Architecture:** The `MaschineMK2_linux` daemon acts as a dumb control surface (pads → NoteOn, buttons/encoders → CC, LEDs ← OSC). A new Python ctrldev driver holds the logic and writes patterns into zynseq, so patterns persist in snapshots and the touchscreen pattern editor mirrors them. Pure logic (euclid, division table, OSC encoding) lives in a separate dependency-free module so it is unit-testable off-Pi.

**Tech Stack:** Python 3 (Zynthian ctrldev framework, ctypes to `libzynseq.so`), stdlib `unittest`, raw UDP OSC, Rust (one small patch to the daemon).

**Spec:** `docs/superpowers/specs/2026-08-06-maschine-drum-rig-design.md`

## Global Constraints

- Python style: PEP 8, 120-char lines, `logging` only — never `print()`.
- The driver module filename and its class name must be identical: the loader does `getattr(module, module_name)` (`zyngine/zynthian_ctrldev_manager.py:95`). Same rule for the helper module.
- Every helper module in `zyngine/ctrldev/` must expose a class named after the file with `dev_ids = []`, otherwise the loader logs an error at every startup.
- The helper module must import nothing from Zynthian — tests import it by path, and `import zyngine` pulls in every engine (`zyngine/__init__.py:29-53`).
- No pytest on the dev machine. Tests use stdlib `unittest`, run with `python3 -m unittest`.
- Local dev repo: `/home/witzman/zynth/zynthian-ui` (git, branch `vangelis`). Runtime on Pi: `/zynthian/zynthian-ui`. Every Pi task starts by copying changed files across.
- **The Pi's installed Zynthian is an older API than the local checkout, and the Pi is authoritative.** Verified on 2026-08-06 against `/zynthian/zynthian-ui`: zynseq addresses sequences as `(bank, sequence, track)` with `self.zynseq.bank` — there is no `scene`/`phrase` level; `getSteps()`, `getClocksPerStep()`, `getStepsPerBeat()`, `setStepsPerBeat()`, `setBeatsInPattern()` and `clear()` all act on the *selected* pattern and take no pattern argument; `getPattern(bank, sequence, track, position)`; `getPlayPosition(bank, sequence)`; `setPlayPosition(bank, sequence, clock)`; `toggleMute(bank, sequence, track)`; `isMuted(bank, sequence, track)`; there is no `clearPattern(index)`. The zynseq subsignal constants live on the zynseq object (`self.zynseq.SS_SEQ_PROGRESS`), not on `zynsigman`. The base ctrldev class does **not** set `self.zynseq` — only its zynpad subclass does. Never write a zynseq call from memory or from the local checkout's header; every signature in this plan was read off the Pi.
- Pi access: `ssh root@<PI_IP>`. `zynthian.local` does not resolve from WSL2 — use the IP. As of 2026-08-06 the last known IP (192.168.2.123) does not respond; confirm the current IP before Task 1.
- Euclid placement must match the daemon's existing algorithm for parity with the published tutorial: hit `i` at `floor(i * steps / hits)` (`MaschineMK2_linux/src/sequencer.rs:1-12`).
- Button CC values: press = 127, release = 0. Act on press only (`cc_math.rs:6`).
- All 8 drum chains use one shared FluidSynth process; per-group filter is CC 74 / CC 71, exposed as the controllers `'filter cutoff'` and `'filter resonance'` (`zynthian_engine_fluidsynth.py:66-67`).

---

## File Structure

**Create:**
- `zyngine/ctrldev/maschine_mk2_lib.py` — pure logic, no Zynthian imports: euclid + rotation, division table, OSC message encoding, LED diff cache.
- `zyngine/ctrldev/tests/test_maschine_mk2_lib.py` — `unittest` tests for the above. Not scanned by the driver loader (its glob is non-recursive, `zynthian_ctrldev_manager.py:76`).
- `zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py` — the driver: MIDI decode, group selection, zynseq writes, transport, mutes, filter, LED render.

**Modify:**
- `MaschineMK2_linux/src/main.rs` — one match arm group so Group A–H emit CC 80–87.
- `~/zynth-docs/htmldoku/project-midi-reference.md` — record the new CC 80–87 assignment and the verified encoder CCs.

All paths are relative to `/home/witzman/zynth/zynthian-ui` unless stated otherwise.

---

## Task 1: Pi reconnaissance — lock the unverified facts

Nothing else can be written safely until three things are known: the ALSA port name the driver matches on, the zynseq call arity, and whether the daemon's pad/button CCs arrive as documented.

**Files:**
- Modify: `~/zynth-docs/htmldoku/project-midi-reference.md`

**Interfaces:**
- Consumes: nothing.
- Produces: three facts used by every later task — `DEV_ID` (exact string for `dev_ids`), `ZYNSEQ_ARITY` (3 or 4 positional args for sequence addressing), and confirmation that Group buttons emit nothing on MIDI.

- [ ] **Step 1: Find the Pi and confirm the daemon is up**

```bash
# If the IP is unknown, ask the user — mDNS does not resolve from WSL2.
PI=192.168.2.123
ssh root@$PI 'systemctl is-active maschine-mk2.service; systemctl is-active zynthian.service'
```

Expected: both print `active`. If `maschine-mk2.service` is inactive, start it with `systemctl start maschine-mk2.service` before continuing.

- [ ] **Step 2: Capture the exact ALSA port name**

```bash
ssh root@$PI 'aconnect -l'
```

Look for the daemon's client (it appears as `maschine.rs`) and note the **exact** port name string Zynthian uses for its MIDI input, e.g. `maschine.rs Pads MIDI`. Zynthian matches `dev_ids` against `zynautoconnect.get_midi_in_devid(izmip)`.

To see the id string Zynthian itself resolved, check the log:

```bash
ssh root@$PI 'journalctl -u zynthian --no-pager | grep -i "ctrldev\|devid" | tail -20'
```

Record the resolved string verbatim. This becomes `DEV_ID`.

- [ ] **Step 3: Confirm pad, F-button and Group CC behaviour**

```bash
ssh root@$PI 'aconnect -l | grep -B2 "Pads MIDI"'   # find the client:port numbers
ssh root@$PI 'timeout 20 amidi -d -p hw:CLIENT,0,0' # substitute the numbers found above
```

While it runs: hit pad 1, then F1, then Group A, then Play.

Expected: pad 1 → `90 30 vv` if Group C is the current base (note 48), F1 → `B0 27 7F` then `B0 27 00` (CC 39), Play → `B0 01 7F`. Group A → **nothing**. Record what actually appears.

If Group A does emit something, the Task 2 Rust patch is unnecessary — note that and skip it.

- [ ] **Step 4: Resolve the zynseq call arity** — *done 2026-08-06, findings are in Global Constraints; re-run only to confirm nothing changed*

```bash
ssh root@$PI 'grep -n "void toggleMute\|bool isMuted\|uint32_t getPlayPosition\|void addPattern\|getSteps\|getClocksPerStep\|getPattern(" /zynthian/zynthian-ui/zynlibs/zynseq/zynseq.h'
ssh root@$PI 'grep -n "toggleMute\|isMuted" /zynthian/zynthian-ui/zyngui/zynthian_gui_arranger.py'
```

The header is the authority for what the compiled `libzynseq.so` expects. Record whether sequence addressing takes `(scene, phrase, sequence, track)` or `(scene, sequence, track)`. This becomes `ZYNSEQ_ARITY`.

Also record whether `getSteps` takes a pattern argument or none — the header declares `getSteps()` with no argument (`zynseq.h:635`) while `akai_apc_key25_mk2.py:2177` calls `getSteps(pattern)`. ctypes will not flag the mismatch. If the installed header has no argument, drop the argument everywhere this plan writes `getSteps(pattern)` and call `selectPattern` first instead.

- [ ] **Step 5: Record the findings in the MIDI reference**

Add to `~/zynth-docs/htmldoku/project-midi-reference.md`, in the Maschine MK2 daemon section, replacing the vague transport row:

```markdown
| Maschine MK2 | Play / Erase / Rec / Grid / Restart | 1 | CC 1 / 2 / 3 / 4 / 7 | driver: transport | Drum Rig | `[verified]` |
| Maschine MK2 | F1–F8 (above displays) | 1 | CC 39–46 | driver: track mute | Drum Rig | `[verified]` |
| Maschine MK2 | Group A–H | 1 | CC 80–87 *(daemon patch)* | driver: group select | Drum Rig | `[draft]` |
| Maschine MK2 | Encoders 1–8 | 1 | CC 16–23 (`encoder_ccs` in `maschine.json`) | driver: euclid + filter | Drum Rig | `[verified]` |
```

- [ ] **Step 6: Regenerate the docs and commit**

```bash
cd ~/zynth-docs
python3 htmldoku/generate-html.py
git add htmldoku/project-midi-reference.md docs/zynthian-Doku/
git commit -m "docs: record verified Maschine MK2 CC map for the drum rig"
```

---

## Task 2: Daemon patch — Group A–H emit CC 80–87

**Files:**
- Modify: `/home/witzman/zynth/MaschineMK2_linux/src/main.rs` (the `"group_a"` … `"group_h"` arms, currently at lines 1087-1142)

**Interfaces:**
- Consumes: nothing.
- Produces: CC 80–87 on ch 1, 127 on press and 0 on release, for Group A–H. Existing note-base and page behaviour unchanged — the driver depends on the note base still moving.

Skip this task if Task 1 Step 3 showed Group buttons already emitting MIDI.

- [ ] **Step 1: Add the CC emission to each group arm**

Each of the eight arms gets one added line. Group A shown; repeat with 81 for `group_b`, 82 for `group_c`, 83 for `group_d`, 84 for `group_e`, 85 for `group_f`, 86 for `group_g`, 87 for `group_h`:

```rust
                "group_a" => {
                    let msg = Message::RPN7(Ch1, 80, cc_math::button_cc_value(is_down));
                    let _ = self.seq_port.send_message(&msg);
                    self.seq_handle.drain_output();
                    if maschine.get_padmode() == 2 && is_down {
                        if maschine.get_mod() == 1 { maschine.apply_euclidean(1); }
                        else { maschine.set_seq_page(0); }
                        self.refresh_seq_page(maschine);
                    } else { maschine.set_midi_note_base(24); }
                }
```

- [ ] **Step 2: Add a unit test for the CC numbering**

The CC numbers are a plain offset from the group index. Put the mapping in a function so it is testable. Add to `src/cc_math.rs`:

```rust
pub fn group_cc(group_idx: usize) -> u16 {
    80 + (group_idx.min(7) as u16)
}
```

and in the `mod tests` block of the same file:

```rust
    #[test]
    fn group_cc_maps_a_to_80_and_h_to_87() {
        assert_eq!(group_cc(0), 80);
        assert_eq!(group_cc(7), 87);
    }

    #[test]
    fn group_cc_clamps_above_h() {
        assert_eq!(group_cc(99), 87);
    }
```

- [ ] **Step 3: Run the test and confirm it fails to compile or fails**

```bash
cd /home/witzman/zynth/MaschineMK2_linux
cargo test cc_math 2>&1 | tail -20
```

Expected before Step 1/2 are complete: compile error `cannot find function group_cc`. After: PASS.

- [ ] **Step 4: Use the helper in the group arms**

Replace the literal `80` … `87` in the eight arms with `cc_math::group_cc(0)` … `cc_math::group_cc(7)` so the numbering has exactly one source.

- [ ] **Step 5: Build and test**

```bash
cd /home/witzman/zynth/MaschineMK2_linux
cargo build --release 2>&1 | tail -5
cargo test 2>&1 | tail -10
```

Expected: build succeeds, all tests pass.

- [ ] **Step 6: Deploy to the Pi and verify on hardware**

```bash
scp target/release/maschine root@$PI:/usr/local/bin/maschine.new
ssh root@$PI 'systemctl stop maschine-mk2.service && mv /usr/local/bin/maschine.new /usr/local/bin/maschine && chmod +x /usr/local/bin/maschine && systemctl start maschine-mk2.service && systemctl is-active maschine-mk2.service'
```

Confirm the binary path first with `ssh root@$PI 'systemctl cat maschine-mk2.service | grep ExecStart'` and use whatever path that shows.

Then dump MIDI while pressing Group A and Group H:

```bash
ssh root@$PI 'timeout 15 amidi -d -p hw:CLIENT,0,0'
```

Expected: `B0 50 7F` / `B0 50 00` for Group A (0x50 = 80), `B0 57 7F` / `B0 57 00` for Group H.

- [ ] **Step 7: Commit**

```bash
cd /home/witzman/zynth/MaschineMK2_linux
git add src/main.rs src/cc_math.rs
git commit -m "feat: emit CC 80-87 for Group A-H buttons

The Zynthian ctrldev driver needs group selection over MIDI. Group
buttons previously only changed the internal note base and page.
Note-base behaviour is unchanged — the driver decodes pad indices
from it."
```

---

## Task 3: Pure logic module — euclid, rotation, division table

**Files:**
- Create: `zyngine/ctrldev/maschine_mk2_lib.py`
- Create: `zyngine/ctrldev/tests/test_maschine_mk2_lib.py`

**Interfaces:**
- Consumes: nothing.
- Produces, all as `@staticmethod` on class `maschine_mk2_lib`:
  - `DIVISIONS` — tuple of `(label: str, steps_per_beat: int, beats: int)`, index 0–4.
  - `step_count(div_idx: int) -> int`
  - `euclid(steps: int, hits: int) -> list[bool]`
  - `rotate(pattern: list[bool], offset: int) -> list[bool]`
  - `build_pattern(div_idx: int, hits: int, rotation: int) -> list[bool]`
  - `clamp_to_steps(value: int, div_idx: int) -> int`

- [ ] **Step 1: Write the failing tests**

Create `zyngine/ctrldev/tests/test_maschine_mk2_lib.py`:

```python
import os
import sys
import unittest

sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))
from maschine_mk2_lib import maschine_mk2_lib as lib  # noqa: E402


class TestDivisions(unittest.TestCase):

    def test_five_divisions_in_order(self):
        labels = [d[0] for d in lib.DIVISIONS]
        self.assertEqual(labels, ["1/32", "1/16", "1/8", "1/16T", "1/8T"])

    def test_straight_divisions_have_16_steps(self):
        self.assertEqual(lib.step_count(0), 16)
        self.assertEqual(lib.step_count(1), 16)
        self.assertEqual(lib.step_count(2), 16)

    def test_triplet_divisions_have_12_steps(self):
        self.assertEqual(lib.step_count(3), 12)
        self.assertEqual(lib.step_count(4), 12)

    def test_steps_per_beat_matches_division(self):
        self.assertEqual(lib.DIVISIONS[0][1], 8)
        self.assertEqual(lib.DIVISIONS[1][1], 4)
        self.assertEqual(lib.DIVISIONS[2][1], 2)
        self.assertEqual(lib.DIVISIONS[3][1], 6)
        self.assertEqual(lib.DIVISIONS[4][1], 3)

    def test_beats_times_spb_equals_step_count(self):
        for idx, (_, spb, beats) in enumerate(lib.DIVISIONS):
            self.assertEqual(spb * beats, lib.step_count(idx))


class TestEuclid(unittest.TestCase):

    def test_zero_hits_is_empty(self):
        self.assertEqual(lib.euclid(16, 0), [False] * 16)

    def test_all_hits_is_full(self):
        self.assertEqual(lib.euclid(16, 16), [True] * 16)

    def test_four_hits_evenly_spaced(self):
        p = lib.euclid(16, 4)
        self.assertEqual([i for i, v in enumerate(p) if v], [0, 4, 8, 12])

    def test_three_hits_matches_daemon_bresenham(self):
        p = lib.euclid(16, 3)
        self.assertEqual([i for i, v in enumerate(p) if v], [0, 5, 10])

    def test_first_hit_always_on_step_zero(self):
        self.assertTrue(lib.euclid(16, 1)[0])

    def test_hits_clamped_to_steps(self):
        self.assertEqual(sum(lib.euclid(12, 20)), 12)


class TestRotate(unittest.TestCase):

    def test_rotation_zero_is_identity(self):
        p = lib.euclid(16, 4)
        self.assertEqual(lib.rotate(p, 0), p)

    def test_rotation_shifts_hits_later(self):
        p = lib.rotate(lib.euclid(16, 4), 1)
        self.assertEqual([i for i, v in enumerate(p) if v], [1, 5, 9, 13])

    def test_rotation_wraps_around(self):
        p = lib.rotate(lib.euclid(16, 4), 16)
        self.assertEqual([i for i, v in enumerate(p) if v], [0, 4, 8, 12])

    def test_rotation_preserves_hit_count(self):
        p = lib.rotate(lib.euclid(12, 5), 7)
        self.assertEqual(sum(p), 5)


class TestBuildPattern(unittest.TestCase):

    def test_build_uses_division_step_count(self):
        self.assertEqual(len(lib.build_pattern(3, 4, 0)), 12)

    def test_build_applies_hits_and_rotation(self):
        p = lib.build_pattern(1, 4, 2)
        self.assertEqual([i for i, v in enumerate(p) if v], [2, 6, 10, 14])


class TestClamp(unittest.TestCase):

    def test_clamp_reduces_value_over_step_count(self):
        self.assertEqual(lib.clamp_to_steps(16, 3), 12)

    def test_clamp_leaves_value_in_range(self):
        self.assertEqual(lib.clamp_to_steps(5, 3), 5)

    def test_clamp_floors_at_zero(self):
        self.assertEqual(lib.clamp_to_steps(-3, 1), 0)


if __name__ == "__main__":
    unittest.main()
```

- [ ] **Step 2: Run the tests and confirm they fail**

```bash
cd /home/witzman/zynth/zynthian-ui
python3 -m unittest discover -s zyngine/ctrldev/tests -v 2>&1 | tail -10
```

Expected: `ModuleNotFoundError: No module named 'maschine_mk2_lib'`.

- [ ] **Step 3: Write the module**

Create `zyngine/ctrldev/maschine_mk2_lib.py`:

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
# Pure helper logic for the Maschine MK2 drum rig ctrldev driver.
#
# Imports nothing from Zynthian on purpose: the unit tests load this module
# directly by path, and importing the zyngine package pulls in every engine.
#
# The class name matches the filename because the ctrldev loader does
# getattr(module, module_name). dev_ids is empty so no device ever matches it.


class maschine_mk2_lib:

    dev_ids = []

    # (label, steps_per_beat, beats) — steps_per_beat * beats == step count
    DIVISIONS = (
        ("1/32", 8, 2),
        ("1/16", 4, 4),
        ("1/8", 2, 8),
        ("1/16T", 6, 2),
        ("1/8T", 3, 4),
    )

    @staticmethod
    def step_count(div_idx):
        """Number of steps a division yields: 16 straight, 12 triplet"""

        _, spb, beats = maschine_mk2_lib.DIVISIONS[div_idx]
        return spb * beats

    @staticmethod
    def euclid(steps, hits):
        """Bresenham placement, identical to the daemon's sequencer.rs:
        hit i lands on floor(i * steps / hits), so hit 0 is always step 0"""

        pattern = [False] * steps
        hits = max(0, min(hits, steps))
        for i in range(hits):
            pattern[(i * steps) // hits] = True
        return pattern

    @staticmethod
    def rotate(pattern, offset):
        """Rotate a pattern forward in time, wrapping at the end"""

        n = len(pattern)
        if n == 0:
            return []
        offset %= n
        return pattern[-offset:] + pattern[:-offset] if offset else list(pattern)

    @staticmethod
    def build_pattern(div_idx, hits, rotation):
        """Full pattern for one group from its three euclid parameters"""

        steps = maschine_mk2_lib.step_count(div_idx)
        return maschine_mk2_lib.rotate(maschine_mk2_lib.euclid(steps, hits), rotation)

    @staticmethod
    def clamp_to_steps(value, div_idx):
        """Clamp a hit count or rotation into a division's step range"""

        return max(0, min(value, maschine_mk2_lib.step_count(div_idx)))
```

- [ ] **Step 4: Run the tests and confirm they pass**

```bash
cd /home/witzman/zynth/zynthian-ui
python3 -m unittest discover -s zyngine/ctrldev/tests -v 2>&1 | tail -5
```

Expected: `OK`, 18 tests.

- [ ] **Step 5: Commit**

```bash
cd /home/witzman/zynth/zynthian-ui
git add zyngine/ctrldev/maschine_mk2_lib.py zyngine/ctrldev/tests/test_maschine_mk2_lib.py
git commit -m "feat: add euclid and division helpers for Maschine drum rig"
```

---

## Task 4: Pure logic module — OSC encoding and LED diff

**Files:**
- Modify: `zyngine/ctrldev/maschine_mk2_lib.py`
- Modify: `zyngine/ctrldev/tests/test_maschine_mk2_lib.py`

**Interfaces:**
- Consumes: `maschine_mk2_lib` from Task 3.
- Produces:
  - `osc_message(path: str, args: list) -> bytes` — OSC 1.0 packet, `int` → `i`, `float` → `f`.
  - `pad_osc(pad: int, color: int, brightness: float) -> bytes`
  - `button_osc(name: str, color: int, brightness: float) -> bytes`
  - `class led_cache` with `changed(key, value) -> bool` and `clear()`.

- [ ] **Step 1: Write the failing tests**

Append to `zyngine/ctrldev/tests/test_maschine_mk2_lib.py`, above the `if __name__` block:

```python
class TestOscMessage(unittest.TestCase):

    def test_path_is_null_terminated_and_padded_to_4(self):
        msg = lib.osc_message("/ab", [])
        self.assertEqual(msg[:4], b"/ab\x00")

    def test_type_tag_for_int_and_float(self):
        msg = lib.osc_message("/x", [1, 2.0])
        self.assertIn(b",if", msg)

    def test_int_argument_is_big_endian(self):
        msg = lib.osc_message("/x", [1])
        self.assertEqual(msg[-4:], b"\x00\x00\x00\x01")

    def test_total_length_is_multiple_of_four(self):
        for path in ("/a", "/abc", "/abcd", "/abcde"):
            self.assertEqual(len(lib.osc_message(path, [3, 0.5])) % 4, 0)

    def test_pad_osc_targets_pad_path_with_three_args(self):
        msg = lib.pad_osc(3, 0xFF8800, 0.7)
        self.assertEqual(msg[:16], b"/maschine/pad\x00\x00\x00")
        self.assertIn(b",iif", msg)

    def test_button_osc_targets_named_button_path(self):
        msg = lib.button_osc("f1", 0xFFFFFF, 1.0)
        self.assertTrue(msg.startswith(b"/maschine/button/f1"))
        self.assertIn(b",if", msg)


class TestLedCache(unittest.TestCase):

    def test_first_write_is_a_change(self):
        cache = lib.led_cache()
        self.assertTrue(cache.changed("pad3", (0xFF0000, 1.0)))

    def test_same_value_twice_is_not_a_change(self):
        cache = lib.led_cache()
        cache.changed("pad3", (0xFF0000, 1.0))
        self.assertFalse(cache.changed("pad3", (0xFF0000, 1.0)))

    def test_different_value_is_a_change(self):
        cache = lib.led_cache()
        cache.changed("pad3", (0xFF0000, 1.0))
        self.assertTrue(cache.changed("pad3", (0xFF0000, 0.1)))

    def test_clear_forgets_everything(self):
        cache = lib.led_cache()
        cache.changed("pad3", (0xFF0000, 1.0))
        cache.clear()
        self.assertTrue(cache.changed("pad3", (0xFF0000, 1.0)))
```

- [ ] **Step 2: Run the tests and confirm they fail**

```bash
cd /home/witzman/zynth/zynthian-ui
python3 -m unittest discover -s zyngine/ctrldev/tests 2>&1 | tail -5
```

Expected: `AttributeError: type object 'maschine_mk2_lib' has no attribute 'osc_message'`.

- [ ] **Step 3: Implement OSC encoding and the LED cache**

Add to `zyngine/ctrldev/maschine_mk2_lib.py` — the `import struct` goes at the top of the file, the `led_cache` class at module level after `maschine_mk2_lib`:

```python
import struct
```

Inside class `maschine_mk2_lib`:

```python
    @staticmethod
    def _osc_pad(data):
        """Pad a byte string to the next multiple of 4"""

        return data + b"\x00" * ((4 - len(data) % 4) % 4)

    @staticmethod
    def osc_message(path, args):
        """Minimal OSC 1.0 encoder. Only int (i) and float (f) are needed —
        the daemon's handler accepts nothing else (main.rs:609-665)"""

        out = maschine_mk2_lib._osc_pad(path.encode("ascii") + b"\x00")
        tags = ","
        body = b""
        for arg in args:
            if isinstance(arg, bool):
                raise TypeError("OSC bool not supported by the daemon")
            if isinstance(arg, int):
                tags += "i"
                body += struct.pack(">i", arg)
            elif isinstance(arg, float):
                tags += "f"
                body += struct.pack(">f", arg)
            else:
                raise TypeError(f"unsupported OSC argument type: {type(arg)}")
        out += maschine_mk2_lib._osc_pad(tags.encode("ascii") + b"\x00")
        return out + body

    @staticmethod
    def pad_osc(pad, color, brightness):
        """LED write for one pad. Pad 0 is bottom-left (daemon commit a42ff17)"""

        return maschine_mk2_lib.osc_message("/maschine/pad", [int(pad), int(color), float(brightness)])

    @staticmethod
    def button_osc(name, color, brightness):
        """LED write for one named button, e.g. 'f1', 'group_a', 'play'"""

        return maschine_mk2_lib.osc_message(
            f"/maschine/button/{name}", [int(color), float(brightness)])
```

At module level:

```python
class led_cache:
    """Remembers the last value written per LED so only changes go on the wire.
    The daemon has already been flooded off the USB bus once by unthrottled
    writes (commit ffc8f2b), so diffing is required, not an optimisation."""

    def __init__(self):
        self._last = {}

    def changed(self, key, value):
        """True if value differs from the last one stored under key"""

        if self._last.get(key) == value:
            return False
        self._last[key] = value
        return True

    def clear(self):
        self._last = {}
```

Expose it on the helper class so tests and the driver reach it the same way — add inside `maschine_mk2_lib`, after the static methods:

```python
    led_cache = None  # bound below to avoid a forward reference
```

and at the end of the file:

```python
maschine_mk2_lib.led_cache = led_cache
```

- [ ] **Step 4: Run the tests and confirm they pass**

```bash
cd /home/witzman/zynth/zynthian-ui
python3 -m unittest discover -s zyngine/ctrldev/tests 2>&1 | tail -5
```

Expected: `OK`, 28 tests.

- [ ] **Step 5: Commit**

```bash
cd /home/witzman/zynth/zynthian-ui
git add zyngine/ctrldev/maschine_mk2_lib.py zyngine/ctrldev/tests/test_maschine_mk2_lib.py
git commit -m "feat: add OSC encoder and LED diff cache for Maschine drum rig"
```

---

## Task 5: Prepared snapshot — 8 drum chains and 8 sequences

Hardware task, no code. The driver reads this structure and never creates it.

**Files:** none — the deliverable is a snapshot on the Pi.

**Interfaces:**
- Consumes: nothing.
- Produces: a snapshot named `maschine-drum-rig` in bank `000`, with chains on MIDI channels 1–8 and 8 zynseq sequences in one scene, each holding one 16-step pattern.

- [ ] **Step 1: Create the 8 drum chains**

On the touchscreen, for each of the 8 groups: tap **+** at the right edge of the Mixer → **Instrument** → **FluidSynth** → pick the GM drum kit preset → open the chain → **Chain Options** → set **MIDI Channel** to 1 for group A, 2 for B, and so on to 8 for H.

**Verify:** the Mixer shows 8 chains, and `Chain Options` reports channels 1–8 with no duplicates.

- [ ] **Step 2: Create one sequence per group**

Open the pattern editor / zynpad and create 8 sequences in the current scene, one per MIDI channel 1–8, each with a single 16-step pattern. Program one hit on step 0 of each so no pattern is empty.

**Verify:** starting transport plays 8 simultaneous hits on the downbeat.

- [ ] **Step 3: Note the zynseq addressing of each sequence**

```bash
ssh root@$PI 'grep -n "def get_pattern\b" -A12 /zynthian/zynthian-ui/zynlibs/zynseq/zynseq.py'
```

Record for each group: scene, phrase (if the installed arity has one), sequence, track, and the pattern id. The driver needs the group → pattern id mapping.

- [ ] **Step 4: Save the snapshot**

Tap **OPT/ADMIN** → **Snapshots** → navigate **into** bank **000** → type `maschine-drum-rig` → tap the checkmark.

Saving at the root level makes the snapshot invisible in the UI — it must go inside `000`.

**Verify:**

```bash
ssh root@$PI 'ls -l /zynthian/zynthian-my-data/snapshots/000/ | grep maschine'
```

Expected: `maschine-drum-rig.zss` listed.

- [ ] **Step 5: Confirm the mapping survives a reload**

Load a different snapshot, then reload `maschine-drum-rig`.

**Verify:** 8 chains on channels 1–8 return and transport still plays 8 simultaneous hits.

---

## Task 6: Driver skeleton — loads, selects groups, toggles steps, renders pads

This is the PoC. It proves daemon → driver → zynseq → sound end to end.

**Files:**
- Create: `zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py`

**Interfaces:**
- Consumes: `maschine_mk2_lib` (Tasks 3–4), `DEV_ID` and `ZYNSEQ_ARITY` (Task 1), the snapshot structure (Task 5), CC 80–87 (Task 2).
- Produces: class `zynthian_ctrldev_maschine_mk2` with `_seq_addr(group)`, `_select_group(group)`, `_toggle_step(step)`, `_render_pads()`, `_send_osc(packet)`.

- [ ] **Step 1: Write the driver**

Create `zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py`. Replace `DEV_ID_FROM_TASK_1` with the string captured in Task 1 Step 2, and set `SEQ_ADDR_HAS_PHRASE` from Task 1 Step 4:

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
# Controller driver for Maschine MK2 via the MaschineMK2_linux daemon.
#
# 8 groups x 16 steps euclidean drum sequencer. All sequencing lives in
# zynseq, so patterns persist in snapshots and the touchscreen pattern
# editor mirrors them.
#
# MIDI in (ch 1, from the daemon):
#   Pads      NoteOn, note = group note base + pad index
#   Group A-H CC 80-87   (requires the daemon's group-CC patch)
#   Encoders  CC 16-20   (set via encoder_ccs in maschine.json)
#   F1-F8     CC 39-46
#   Play 1 - Erase 2 - Restart 7
#
# LED out: OSC to the daemon on 127.0.0.1:42434 (main.rs:609-665)

import logging
import socket

from zyngine.ctrldev.zynthian_ctrldev_base import zynthian_ctrldev_base
from zyngine.ctrldev.maschine_mk2_lib import maschine_mk2_lib as lib

OSC_ADDR = ("127.0.0.1", 42434)

GROUP_CC_FIRST = 80                 # Group A..H = CC 80..87
GROUP_NOTE_BASE = (24, 36, 48, 60, 72, 84, 96, 108)

COLOR_STEP_ON = 0xFF8800
COLOR_STEP_OFF = 0x101010
BRIGHT_ON = 0.9
BRIGHT_OFF = 0.05

# Task 1 findings
DEV_ID = "DEV_ID_FROM_TASK_1"


class zynthian_ctrldev_maschine_mk2(zynthian_ctrldev_base):

    dev_ids = [DEV_ID]
    driver_name = "Maschine MK2 Drum Rig"
    driver_description = "8 groups x 16 steps euclidean drum sequencer on zynseq"
    unroute_from_chains = True      # pads must not reach chains directly

    def __init__(self, state_manager, idev_in, idev_out=None):
        super().__init__(state_manager, idev_in, idev_out)
        # The installed base class sets self.zynseq only in its zynpad
        # subclass, so this driver wires it up itself.
        self.zynseq = state_manager.zynseq
        self.libseq = self.zynseq.libseq
        self.group = 0                       # selected group, 0 = A
        self.leds = lib.led_cache()
        self.osc = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # --- plumbing ------------------------------------------------------

    def _send_osc(self, packet):
        try:
            self.osc.sendto(packet, OSC_ADDR)
        except OSError as e:
            logging.error(f"Maschine OSC send failed: {e}")

    def _seq_addr(self, group):
        """Sequence address for a group, as the installed libzynseq expects:
        (bank, sequence, track). Every zynseq call routes through here."""

        return (self.zynseq.bank, group, 0)

    def _pattern_of(self, group):
        """Pattern id backing a group, read from zynseq (not cached).
        Installed signature: getPattern(bank, sequence, track, position)"""

        return self.libseq.getPattern(self.zynseq.bank, group, 0, 0)

    def _select_pattern(self, group):
        """Select a group's pattern and return its id. The installed API is
        selection-based — getSteps(), setStepsPerBeat(), setBeatsInPattern()
        and clear() all act on the selected pattern and take no pattern
        argument, so every read or write is preceded by this call."""

        pattern = self._pattern_of(group)
        self.libseq.selectPattern(pattern)
        return pattern

    # --- lifecycle -----------------------------------------------------

    def init(self):
        super().init()
        self._render_all()

    def end(self):
        self.light_off()
        super().end()

    def light_off(self):
        self.leds.clear()
        for pad in range(16):
            self._send_osc(lib.pad_osc(pad, COLOR_STEP_OFF, 0.0))
        for group in range(8):
            self._send_osc(lib.button_osc(f"group_{chr(ord('a') + group)}", 0xFFFFFF, 0.0))

    # --- MIDI ----------------------------------------------------------

    def midi_event(self, ev):
        evtype = (ev[0] >> 4) & 0x0F

        if evtype == 0x9 and ev[2] > 0:          # NoteOn, ignore note-off
            step = ev[1] - GROUP_NOTE_BASE[self.group]
            if 0 <= step < 16:
                self._toggle_step(step)
                return True
            return False

        if evtype == 0xB:
            cc_num, cc_val = ev[1] & 0x7F, ev[2] & 0x7F
            if cc_val != 127:                    # act on press only
                return False
            group = cc_num - GROUP_CC_FIRST
            if 0 <= group < 8:
                self._select_group(group)
                return True

        return False

    # --- actions -------------------------------------------------------

    def _select_group(self, group):
        self.group = group
        self._render_all()

    def _toggle_step(self, step):
        self._select_pattern(self.group)
        steps = self.libseq.getSteps()
        if step >= steps:
            return
        note = self._group_note(self.group)
        if self.libseq.getNoteVelocity(step, note):
            self.libseq.removeNote(step, note)
        else:
            self.libseq.addNote(step, note, 100, 1.0, 0.0)
        self.libseq.updateSequenceInfo()
        self._render_pads()

    def _group_note(self, group):
        """The single note a group's pattern uses. Read from the pattern's
        first event so the drum kit mapping lives in the snapshot, not here.
        Falls back to the GM note range base when the pattern is empty."""

        pattern = self._pattern_of(group)
        note = self.libseq.getNoteAtIndex(pattern, 0)
        return note if 0 < note < 128 else 36 + group

    # --- LEDs ----------------------------------------------------------

    def _render_pads(self):
        self._select_pattern(self.group)
        steps = self.libseq.getSteps()
        note = self._group_note(self.group)
        for pad in range(16):
            if pad >= steps:
                state = (COLOR_STEP_OFF, 0.0)
            elif self.libseq.getNoteVelocity(pad, note):
                state = (COLOR_STEP_ON, BRIGHT_ON)
            else:
                state = (COLOR_STEP_OFF, BRIGHT_OFF)
            if self.leds.changed(f"pad{pad}", state):
                self._send_osc(lib.pad_osc(pad, state[0], state[1]))

    def _render_groups(self):
        for group in range(8):
            state = (0xFFFFFF, 1.0 if group == self.group else 0.05)
            key = f"group{group}"
            if self.leds.changed(key, state):
                self._send_osc(lib.button_osc(
                    f"group_{chr(ord('a') + group)}", state[0], state[1]))

    def _render_all(self):
        self._render_groups()
        self._render_pads()

    def refresh(self):
        super().refresh()
        self._render_all()
```

- [ ] **Step 2: Confirm the module imports cleanly**

```bash
cd /home/witzman/zynth/zynthian-ui
python3 -c "import ast,sys; ast.parse(open('zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py').read()); print('syntax OK')"
python3 -m unittest discover -s zyngine/ctrldev/tests 2>&1 | tail -3
```

Expected: `syntax OK` and the Task 3/4 tests still `OK`. A full import needs the Zynthian runtime, so it is only exercised on the Pi.

- [ ] **Step 3: Deploy to the Pi**

```bash
cd /home/witzman/zynth/zynthian-ui
scp zyngine/ctrldev/maschine_mk2_lib.py zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py \
    root@$PI:/zynthian/zynthian-ui/zyngine/ctrldev/
ssh root@$PI 'systemctl restart zynthian.service'
```

- [ ] **Step 4: Confirm the driver loaded**

```bash
ssh root@$PI 'journalctl -u zynthian --no-pager | grep -i maschine | tail -20'
```

Expected: a line containing `Loaded ctrldev driver 'zynthian_ctrldev_maschine_mk2'`.

If instead nothing appears, `dev_ids` does not match — re-check Task 1 Step 2 and correct `DEV_ID`. If a class-not-found error appears for `maschine_mk2_lib`, its `dev_ids = []` attribute is missing.

- [ ] **Step 5: Verify on hardware**

Load the `maschine-drum-rig` snapshot. Make sure the MK2 is in normal pad mode (not the daemon's sequencer mode).

Press Group A, tap pads 1, 5, 9, 13. Then press Group B and tap pads 3 and 11.

**Verify:** the tapped pads light up; the touchscreen pattern editor for channel 1 shows hits on steps 0, 4, 8, 12 and for channel 2 on steps 2 and 10; pressing Group A again shows group A's pads lit, not group B's.

Also check for duplicate MIDI events here — the SMC-PAD turned out to have a mirrored port that fired every action twice. A tapped step that immediately untoggles itself is the symptom:

```bash
ssh root@$PI 'aconnect -l | grep -B3 -A3 maschine'
```

Expected: only one connection from the daemon into ZynMidiRouter. If two ports of the daemon are both connected, disable the redundant one in webconf → **Interface** → **MIDI Options** → **MIDI Devices**.

- [ ] **Step 6: Commit**

```bash
cd /home/witzman/zynth/zynthian-ui
git add zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py
git commit -m "feat: add Maschine MK2 drum rig ctrldev driver skeleton

Group A-H select, 16 pads toggle steps of the selected group in zynseq,
pad and group LEDs render over OSC with diffing."
```

---

## Task 7: Transport and playhead

**Files:**
- Modify: `zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py`

**Interfaces:**
- Consumes: Task 6's driver, `_render_pads`, `_seq_addr`.
- Produces: `_playhead()` returning the current step index of the selected group's pattern, plus Play and Restart handling. `_render_pads` gains a playhead overlay.

- [ ] **Step 1: Add the transport constants and handlers**

Add near the other constants:

```python
CC_PLAY = 1
CC_ERASE = 2
CC_RESTART = 7

COLOR_PLAYHEAD = 0xFFFFFF
BRIGHT_PLAYHEAD = 1.0
```

Extend the CC branch of `midi_event`, before the group check:

```python
            if cc_num == CC_PLAY:
                self.state_manager.send_cuia("TOGGLE_PLAY")
                return True
            if cc_num == CC_RESTART:
                for group in range(8):
                    # Installed signature: setPlayPosition(bank, sequence, clock)
                    self.libseq.setPlayPosition(self.zynseq.bank, group, 0)
                return True
```

- [ ] **Step 2: Add the playhead read**

`getPatternPlayhead()` does not work from a ctrldev driver — the note at `akai_apc_key25_mk2.py:2182` is explicit. Use clocks instead:

```python
    def _playhead(self):
        """Current step of the selected group's pattern, or None when stopped.
        getPatternPlayhead() is unreliable from a ctrldev driver (see the note
        at apc_key25_mk2.py:2182), so derive it from clocks instead.
        Installed signatures: getClocksPerStep() takes no argument and acts on
        the selected pattern; getPlayPosition(bank, sequence)."""

        self._select_pattern(self.group)
        cps = self.libseq.getClocksPerStep()
        if cps <= 0:
            return None
        playpos = self.libseq.getPlayPosition(self.zynseq.bank, self.group)
        if playpos < 0:
            return None
        return playpos // cps
```

- [ ] **Step 3: Overlay the playhead in `_render_pads`**

Replace the body of the pad loop in `_render_pads` with:

```python
        head = self._playhead()
        for pad in range(16):
            if pad >= steps:
                state = (COLOR_STEP_OFF, 0.0)
            elif pad == head:
                state = (COLOR_PLAYHEAD, BRIGHT_PLAYHEAD)
            elif self.libseq.getNoteVelocity(pad, note):
                state = (COLOR_STEP_ON, BRIGHT_ON)
            else:
                state = (COLOR_STEP_OFF, BRIGHT_OFF)
            if self.leds.changed(f"pad{pad}", state):
                self._send_osc(lib.pad_osc(pad, state[0], state[1]))
```

- [ ] **Step 4: Drive the playhead render from the zynseq signal**

Add to `init`, after `super().init()`:

```python
        zynsigman.register_queued(
            zynsigman.S_STEPSEQ, self.zynseq.SS_SEQ_PROGRESS, self._on_progress)
```

and the matching unregister at the top of `end`:

```python
        zynsigman.unregister(
            zynsigman.S_STEPSEQ, self.zynseq.SS_SEQ_PROGRESS, self._on_progress)
```

The subsignal constants live on the zynseq object, not on `zynsigman` — the installed base class does exactly this at `zyngine/ctrldev/zynthian_ctrldev_base.py:208`. `SS_SEQ_PROGRESS` is defined at `zynlibs/zynseq/zynseq.py:85`.

The callback goes through a wrapper because signal handlers are called with the signal's own arguments, which `_render_pads` does not take:

```python
    def _on_progress(self, *args, **kwargs):
        self._render_pads()
```

with the import:

```python
from zyngine.zynthian_signal_manager import zynsigman
```

Already verified on the Pi: `S_STEPSEQ = 9` in `zyngine/zynthian_signal_manager.py:46`, and `SS_SEQ_PLAY_STATE = 1`, `SS_SEQ_REFRESH = 2`, `SS_SEQ_PROGRESS = 3` in `zynlibs/zynseq/zynseq.py:83-85`. `SS_SEQ_PROGRESS` is emitted from `zynseq.py:168`.

- [ ] **Step 5: Deploy and verify**

```bash
cd /home/witzman/zynth/zynthian-ui
scp zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py root@$PI:/zynthian/zynthian-ui/zyngine/ctrldev/
ssh root@$PI 'systemctl restart zynthian.service'
```

Load the snapshot, program hits in two groups, press **Play**.

**Verify:** all 8 groups play together; a white pad sweeps left-to-right in step with the audio; pressing **Play** again stops; **Restart** jumps the pattern back to step 0 without stopping.

- [ ] **Step 6: Commit**

```bash
cd /home/witzman/zynth/zynthian-ui
git add zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py
git commit -m "feat: add transport and playhead LED to Maschine drum rig"
```

---

## Task 8: Euclid encoders

**Files:**
- Modify: `zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py`

**Interfaces:**
- Consumes: `lib.build_pattern`, `lib.step_count`, `lib.clamp_to_steps`, `lib.DIVISIONS`, Task 6's `_select_pattern` / `_group_note`.
- Produces: `_write_pattern(group)`, `_derive_params(group)`, per-group `hits` / `div` / `rot` state.

- [ ] **Step 1: Set the encoder CC numbers on the daemon**

Open the daemon's web editor at `http://$PI:9000` and set encoders 1–5 to CC 16, 17, 18, 19, 20. Confirm they persist:

```bash
ssh root@$PI 'grep -i encoder /zynthian/*/maschine.json 2>/dev/null || find / -name maschine.json 2>/dev/null | head -3'
ssh root@$PI 'timeout 15 amidi -d -p hw:CLIENT,0,0'   # turn encoders 1-5
```

Expected: `B0 10 vv` … `B0 14 vv` as encoders 1–5 move.

- [ ] **Step 2: Add per-group euclid state and the CC handlers**

Add constants:

```python
CC_ENC_HITS = 16
CC_ENC_DIV = 17
CC_ENC_ROT = 18
```

In `__init__`:

```python
        self.hits = [0] * 8
        self.div = [1] * 8               # index into lib.DIVISIONS, 1 = 1/16
        self.rot = [0] * 8
```

In the CC branch of `midi_event`, replace the `if cc_val != 127: return False` guard so encoders are handled before the press-only filter:

```python
            if cc_num in (CC_ENC_HITS, CC_ENC_DIV, CC_ENC_ROT):
                self._encoder(cc_num, cc_val)
                return True
            if cc_val != 127:            # buttons: act on press only
                return False
```

- [ ] **Step 3: Implement the encoder logic**

The daemon sends absolute 0–127 encoder positions (`cc_math.rs:1-4`), so scale rather than accumulate:

```python
    def _encoder(self, cc_num, cc_val):
        group = self.group
        if cc_num == CC_ENC_DIV:
            self.div[group] = min(len(lib.DIVISIONS) - 1, cc_val * len(lib.DIVISIONS) // 128)
            self.hits[group] = lib.clamp_to_steps(self.hits[group], self.div[group])
            self.rot[group] = lib.clamp_to_steps(self.rot[group], self.div[group])
        else:
            steps = lib.step_count(self.div[group])
            value = cc_val * (steps + 1) // 128
            if cc_num == CC_ENC_HITS:
                self.hits[group] = min(steps, value)
            else:
                self.rot[group] = min(steps - 1, value)
        self._write_pattern(group)

    def _write_pattern(self, group):
        """Regenerate a group's whole pattern from its euclid parameters.
        Destructive by design: enc 1-3 own the steps, pad taps are edits
        that the next encoder turn wipes."""

        div_idx = self.div[group]
        label, spb, beats = lib.DIVISIONS[div_idx]
        self._select_pattern(group)
        note = self._group_note(group)

        # All three act on the selected pattern and take no pattern argument.
        # There is no clearPattern(index) in the installed API — clear() is it.
        self.libseq.setBeatsInPattern(beats)
        self.libseq.setStepsPerBeat(spb)
        self.libseq.clear()
        for step, on in enumerate(lib.build_pattern(div_idx, self.hits[group], self.rot[group])):
            if on:
                self.libseq.addNote(step, note, 100, 1.0, 0.0)
        self.libseq.updateSequenceInfo()
        logging.debug(f"Maschine group {group}: {label} hits={self.hits[group]} rot={self.rot[group]}")
        self._render_pads()
```

`setStepsPerBeat` applies to the selected pattern and rescales existing notes (`pattern.cpp:665-681`), which is why the pattern is cleared and rewritten rather than edited in place.

- [ ] **Step 4: Derive hits and division on group select**

Replace `_select_group` from Task 6:

```python
    def _select_group(self, group):
        self.group = group
        self._derive_params(group)
        self._render_all()

    def _derive_params(self, group):
        """Read back what zynseq actually holds so the encoders resume from
        real values after a snapshot load. Rotation is not recoverable — it
        stays at whatever the driver last set."""

        self._select_pattern(group)
        spb = self.libseq.getStepsPerBeat()
        for idx, (_, div_spb, _) in enumerate(lib.DIVISIONS):
            if div_spb == spb:
                self.div[group] = idx
                break
        note = self._group_note(group)
        steps = self.libseq.getSteps()
        self.hits[group] = sum(
            1 for step in range(steps) if self.libseq.getNoteVelocity(step, note))
```

- [ ] **Step 5: Deploy and verify**

```bash
cd /home/witzman/zynth/zynthian-ui
scp zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py root@$PI:/zynthian/zynthian-ui/zyngine/ctrldev/
ssh root@$PI 'systemctl restart zynthian.service'
```

Press Group A, press Play, turn encoder 1 up slowly.

**Verify:** kick density rises from 1 hit to 16 as encoder 1 sweeps; pad LEDs follow every turn; encoder 3 slides the hits later while the count stays the same; encoder 2 at the 1/16T position leaves pads 13–16 dark and the group audibly runs in triplets against the others; switching to Group B and back leaves group A's pattern intact and encoder 1 no longer jumps.

- [ ] **Step 6: Commit**

```bash
cd /home/witzman/zynth/zynthian-ui
git add zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py
git commit -m "feat: add euclid encoders to Maschine drum rig

Enc 1 hits, enc 2 division (incl. triplets at 12 steps), enc 3 rotation.
Params are derived back out of zynseq on group select."
```

---

## Task 9: Mutes, filter, pad preview and Erase

**Files:**
- Modify: `zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py`

**Interfaces:**
- Consumes: Task 8's driver.
- Produces: `_toggle_mute(group)`, `_render_mutes()`, `_set_filter(cc_num, cc_val)`, `_preview(note)`, `_clear_group()`.

- [ ] **Step 1: Add the constants**

```python
CC_F1 = 39                       # F1..F8 = CC 39..46
CC_ENC_CUTOFF = 19
CC_ENC_RESONANCE = 20

F_BUTTON_NAMES = ("f1", "f2", "f3", "f4", "f5", "f6", "f7", "f8")
```

- [ ] **Step 2: Handle the new CCs**

In the CC branch of `midi_event`, add the filter encoders alongside the euclid encoders:

```python
            if cc_num in (CC_ENC_CUTOFF, CC_ENC_RESONANCE):
                self._set_filter(cc_num, cc_val)
                return True
```

and after the press-only guard:

```python
            fbtn = cc_num - CC_F1
            if 0 <= fbtn < 8:
                self._toggle_mute(fbtn)
                return True
            if cc_num == CC_ERASE:
                self._clear_group()
                return True
```

- [ ] **Step 3: Implement mutes**

```python
    def _toggle_mute(self, group):
        self.libseq.toggleMute(*self._seq_addr(group))
        self._render_mutes()

    def _render_mutes(self):
        for group in range(8):
            muted = bool(self.libseq.isMuted(*self._seq_addr(group)))
            state = (0xFFFFFF, 0.0 if muted else 1.0)
            if self.leds.changed(f"mute{group}", state):
                self._send_osc(lib.button_osc(F_BUTTON_NAMES[group], state[0], state[1]))
```

Add `self._render_mutes()` to `_render_all`.

- [ ] **Step 4: Implement the filter encoders**

```python
    def _set_filter(self, cc_num, cc_val):
        """Drive the selected group's chain filter through its zctrl rather
        than raw MIDI, so the touchscreen knobs move too."""

        symbol = "filter cutoff" if cc_num == CC_ENC_CUTOFF else "filter resonance"
        chain_ids = self.chain_manager.get_chain_ids_by_midi_chan(self.group)
        if not chain_ids:
            logging.debug(f"Maschine: no chain on MIDI chan {self.group}")
            return
        chain = self.chain_manager.chains[chain_ids[0]]
        proc = chain.get_processors()[0] if chain.get_processors() else None
        if proc is None or symbol not in proc.controllers_dict:
            logging.debug(f"Maschine: '{symbol}' not found on chain {chain_ids[0]}")
            return
        proc.controllers_dict[symbol].midi_control_change(cc_val)
```

`get_chain_ids_by_midi_chan` takes a 0-indexed channel, so group 0 (A) is channel 0 = the chain shown as MIDI channel 1 in the UI.

- [ ] **Step 5: Implement pad preview and Erase**

The driver claims the port exclusively (`unroute_from_chains = True`), so pads make no sound by themselves. Send the note yourself. Extend `_toggle_step` with a preview after the zynseq write:

```python
        self._preview(note)
```

and add:

```python
    def _preview(self, note):
        """Audible feedback for a pad tap: play the group's drum note once"""

        lib_zyncore.ui_send_note_on(self.group, note, 100)
        lib_zyncore.ui_send_note_off(self.group, note, 0)

    def _clear_group(self):
        self._select_pattern(self.group)
        self.libseq.clear()             # acts on the selected pattern
        self.libseq.updateSequenceInfo()
        self.hits[self.group] = 0
        self._render_pads()
```

with the import:

```python
from zyncoder.zyncore import lib_zyncore
```

**The note-send call above is a placeholder and must be replaced with a verified name before this method is written.** The Pi's `zyncoder/zyncore.py` contains no `note_on` or `note_off` wrapper at all, and the only note sends in the installed ctrldev drivers are `lib_zyncore.dev_send_note_on(idev, chan, note, vel)` — which sends *to* a device for LED feedback, not into the chains. Find the real injection path:

```bash
ssh root@$PI 'grep -rn "write_zynmidi\|zynmidi_send\|send_note_on" /zynthian/zynthian-ui/zyncoder/ /zynthian/zynthian-ui/zyngui/zynthian_gui_pated*.py /zynthian/zynthian-ui/zynlibs/zynseq/zynseq.py | head -20'
```

The pattern editor plays a preview note when a step is selected, so whatever call it uses is the one to copy — use that name and argument order verbatim.

If no usable injection call exists, drop the preview instead of inventing one. It is a nice-to-have, and the obvious workaround (setting `unroute_from_chains = False`) is worse than silence: every pad tap would also trigger group A's chain directly, on every group.

- [ ] **Step 6: Deploy and verify**

```bash
cd /home/witzman/zynth/zynthian-ui
scp zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py root@$PI:/zynthian/zynthian-ui/zyngine/ctrldev/
ssh root@$PI 'systemctl restart zynthian.service'
```

**Verify:** with transport running, F1 silences the kick and its LED goes dark; F1 again brings it back; the mute works while Group C is selected, proving it is selection-independent; encoder 4 sweeps the selected group's cutoff and the touchscreen cutoff knob moves with it; encoder 5 does the same for resonance; tapping a pad plays the drum sound once; Erase clears the selected group's pattern and its pads go dim.

- [ ] **Step 7: Commit**

```bash
cd /home/witzman/zynth/zynthian-ui
git add zyngine/ctrldev/zynthian_ctrldev_maschine_mk2.py
git commit -m "feat: add mutes, filter encoders, pad preview and clear to drum rig"
```

---

## Task 10: Snapshot round-trip and tutorial page

**Files:**
- Create: `~/zynth-docs/htmldoku/project-maschine-drum-rig.md`
- Modify: `~/zynth-docs/htmldoku/generate-html.py` (sidebar entry under Personal Projects)
- Modify: `~/zynth-docs/MD/inwork.md`, `~/zynth-docs/MD/todo.md`

**Interfaces:**
- Consumes: the finished driver.
- Produces: a published tutorial page and updated tracking files.

- [ ] **Step 1: Verify the snapshot round-trip**

Build a full 8-group pattern set with mixed divisions and two groups muted. Save the snapshot over `maschine-drum-rig`. Load a different snapshot, then load `maschine-drum-rig` again.

**Verify:** every group's steps return, the triplet groups still show 12 steps, the muted groups' F-button LEDs are still dark, and turning encoder 1 on a group starts from that group's actual hit count rather than jumping.

If encoder 1 jumps, `_derive_params` is not being called on snapshot load — register it on the `SS_LOAD_SNAPSHOT` signal the base class already listens to via `refresh()`, and derive for all 8 groups there.

- [ ] **Step 2: Write the tutorial page**

Create `~/zynth-docs/htmldoku/project-maschine-drum-rig.md` following the standard structure from `MD/agent-behavior.md`: `# Title`, then **Goal** / **Prerequisites** / **Access**, then `## Part N — …` with `[draft]` tags, `### Step N — …` steps, and a **Verify:** line closing every part.

Parts, mirroring this plan's Pi-verified tasks:

1. Build the 8 drum chains and the prepared snapshot (Task 5)
2. Install the driver and the patched daemon; confirm group select and step toggling (Tasks 2, 6)
3. Transport and playhead (Task 7)
4. Euclid encoders, including the 12-step triplet behaviour (Task 8)
5. Mutes, filter encoders, pad preview (Task 9)

Mark each part `[verified]` only for the parts actually confirmed on the Pi during this plan.

- [ ] **Step 3: Add the sidebar entry**

Add `project-maschine-drum-rig` to the Personal Projects section of the sidebar list in `~/zynth-docs/htmldoku/generate-html.py`, next to the existing `project-maschine-step-sequencer` entry.

- [ ] **Step 4: Generate and commit the docs**

The generator rewrites every page's sidebar, so the whole output directory must be committed:

```bash
cd ~/zynth-docs
python3 htmldoku/generate-html.py
git add htmldoku/project-maschine-drum-rig.md htmldoku/generate-html.py docs/zynthian-Doku/
git commit -m "docs: add tutorial — Maschine MK2 Drum Rig"
```

- [ ] **Step 5: Update the tracking files**

In `~/zynth-docs/MD/inwork.md`, add under Tutorials:

```markdown
- [~] **Maschine MK2 Drum Rig** — 8 groups × 16 steps euclidean drum sequencer via new ctrldev driver; zynseq holds patterns
```

In `~/zynth-docs/MD/todo.md`, add an Active entry listing whichever parts remain unverified, plus these two follow-ups surfaced during design and deliberately left out of scope:

```markdown
- [ ] **Maschine drum rig follow-ups**
  - [ ] Driver should auto-provision the 8 sequences instead of requiring a prepared snapshot
  - [ ] Rotation is not persisted across snapshot reloads — decide whether to store it in the driver state dict (`get_state`/`set_state`)
```

- [ ] **Step 6: Commit**

```bash
cd ~/zynth-docs
git add MD/inwork.md MD/todo.md
git commit -m "docs: track Maschine drum rig tutorial and follow-ups"
```

---

## Deferred to sub-project 2

Two Turing-machine voices controlled from the SMC-PAD, each with filter and reverb/delay sends. Own spec, own plan, after this rig is verified.
