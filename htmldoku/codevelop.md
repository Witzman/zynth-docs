# Co-developing Zynthian with Claude

This page describes the workflow used to develop and maintain this Zynthian project collaboratively with Claude Code. The same pattern applies to working on any of the three repositories.

---

## Project Layout

```
~/zynth/                       ← project root (NOT a git repo)
    zynthian-sys/              ← system scripts, config, hardware detection
    zynthian-ui/               ← Python UI + synth engines
    zynthian-webconf/          ← web configuration interface
    MD/   →  symlink           ← ~/zynth-docs/MD/
    CLAUDE.md  →  symlink      ← ~/zynth-docs/MD/CLAUDE.md

~/zynth-docs/                  ← git repo: this documentation
    MD/                        ← session tracking files
    htmldoku/                  ← documentation source (.md files)
    docs/zynthian-Doku/        ← rendered HTML
```

The symlinks ensure `CLAUDE.md` and `MD/` are visible from `~/zynth/` regardless of which repo directory is active. Claude Code reads `CLAUDE.md` automatically when opened from `~/zynth/`.

---

## Session Workflow

### 1 — Orient

Claude reads `CLAUDE.md` automatically. Also read `MD/inwork.md` at session start. This takes under 10 seconds and prevents Claude from proposing work that is already done or conflicting with in-progress items.

At session start Claude groups `inwork.md` items by readiness:
- `[~]` in development
- `[t]` in testing on Pi
- `[>]` needs PR

Select one item before proceeding.

### 2 — Discover

Before touching any code, read the relevant `htmldoku/` page. This prevents proposing changes that conflict with documented behavior. Then `grep` the relevant `zyngui/` or `zyngine/` file for the actual implementation.

```bash
# Find where an engine is implemented
grep -l "class zynthian_engine_fluidsynth" ~/zynth/zynthian-ui/zyngine/*.py

# Find a specific function
grep -n "def get_preset_list" ~/zynth/zynthian-ui/zyngine/zynthian_engine_fluidsynth.py
```

### 3 — Consult

Before implementing, Claude presents a sign-off card:

```
Task:           <what we're doing>
Files affected: <list>
Test method:    SSH verify | webconf check | both
Proceed? ✓ / ✗
```

This is the right moment to redirect if the scope is wrong.

### 4 — Implement

One task per session. Log adjacent bugs in `MD/bugs.md` and finish the current task first.

Python rules: PEP 8, 120-char line limit, no `print()` (use `logging`), synth engines inherit from `zynthian_engine` base class, GUI screens inherit from `zynthian_gui_base`.

### 5 — Test

```bash
# Restart the changed service on Pi
ssh root@zynthian.local "systemctl restart zynthian"
ssh root@zynthian.local "systemctl restart zynthian-webconf"

# Check status
ssh root@zynthian.local "systemctl status zynthian --no-pager"
ssh root@zynthian.local "journalctl -u zynthian -n 30 --no-pager"

# Check JACK is still healthy
ssh root@zynthian.local "systemctl status jack2 --no-pager"
```

Manual verify: open `http://zynthian.local` and exercise the changed feature.

### 6 — Wrap Up

- Move item in `MD/inwork.md` to `MD/done.md`
- Update `MD/bugs.md` / `MD/decisions.md` if needed
- If documentation needs updating: edit `htmldoku/*.md`, run the generator, commit both

```bash
cd ~/zynth-docs
# edit htmldoku/somefile.md
python3 htmldoku/generate-html.py
git add htmldoku/somefile.md docs/zynthian-Doku/somefile.html
git commit -m "docs: ..."
```

---

## Documentation Generation

```bash
cd ~/zynth-docs
python3 htmldoku/generate-html.py
# → generates docs/zynthian-Doku/*.html
# → copies style.css, search.js, ui.js
# → builds search-index.json
# → copies readme.html → index.html
```

Open `docs/zynthian-Doku/index.html` in a browser to preview locally.

To add a new documentation page:

1. Create `htmldoku/newpage.md`
2. Add an entry to `SIDEBAR` in `htmldoku/generate-html.py`
3. Run the generator
4. Add links from related pages to `newpage.md`

---

## MD/ Tracking Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Always-loaded session primer. Contains project layout, key files, documentation lookup, session workflow. |
| `inwork.md` | Sprint board — active tasks with `[~]`/`[t]`/`[>]` status. |
| `bugs.md` | Known bugs with priority. Format: `## BUG-NNN — title`. |
| `decisions.md` | Architecture Decision Records. Format: `## ADR-NNN — title`. |
| `todo.md` | Backlog items. |
| `done.md` | Completed items (moved from inwork). |
| `sparring.md` | Design review protocol — read before first Python task of each session. |
| `commands.md` | Shell command reference for SSH operations. |

---

## Confidence Markers

Documentation pages can mark uncertain content with `[low]`:

```markdown
The reverb plugin requires at least 256MB free RAM to run smoothly. [low]
```

The generator renders `[low]` text in a muted style. Use it for:
- Wiki-derived information not verified against the code
- Version-dependent behavior
- Hardware-specific observations from a single device

---

## Sparring Protocol

For tasks touching Python code (engines, GUI screens), read `MD/sparring.md` at session start. It defines the layer taxonomy and design principles checklist used to review proposed changes before implementing.

Key layers:
- **Config** — `zynthian_envars.sh`, wiring profiles
- **Script** — `zynthian-sys/sbin/`
- **Engine** — `zyngine/zynthian_engine_*.py`
- **GUI** — `zyngui/zynthian_gui_*.py`
- **Webconf** — `zynthian-webconf/lib/*_handler.py`

Changes should stay within one layer. Cross-layer changes need explicit justification in `decisions.md`.

---

## What's Next

- [Architecture](architecture.md) — system layout and boot sequence
- [Getting Started](getting-started.md) — first hardware setup
- [Webconf Reference](webconf.md) — configuration interface

---

*Version: 2026-05-25*
