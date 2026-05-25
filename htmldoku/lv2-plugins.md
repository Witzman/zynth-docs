# LV2 Plugins

LV2 is a standard plugin format for Linux audio. Zynthian runs LV2 plugins as synth engines or effects via the `jalv` host. This page covers installing, managing, and using LV2 plugins.

---

## How Zynthian Uses LV2

Zynthian wraps each LV2 plugin in `zynthian_engine_jalv.py`. When you select an LV2 plugin as a chain engine, Zynthian:

1. Starts `jalv` as a subprocess with the plugin URI
2. Connects jalv's MIDI input to the chain's MIDI channel
3. Connects jalv's audio output to the JACK audio mixer
4. Exposes the plugin's LV2 ports as Zynthian controls (encoders/knobs)

LV2 plugins include both **instruments** (synthesizers, samplers) and **effects** (reverb, EQ, compressor).

---

## What Plugins Are Installed?

```bash
# List all installed LV2 bundles
ls /usr/lib/lv2/

# List individual plugins with full names
jalv --list | sort
```

The webconf Engines page shows all detected plugins. If a plugin is installed but not visible, regenerate the cache: **Engines** → **Regenerate LV2 Cache**.

The LV2 search path (from `zynthian_envars.sh`):

```bash
/usr/lib/lv2
/usr/lib/arm-linux-gnueabihf/lv2
/usr/local/lib/lv2
/zynthian/zynthian-plugins/lv2
/zynthian/zynthian-data/presets/lv2
/zynthian/zynthian-my-data/presets/lv2
```

---

## Installing LV2 Plugins

### From the Zynthian OS Package Repositories

ZynthianOS is based on Debian. Many LV2 plugins are available as packages:

```bash
# Search for LV2 packages
apt search lv2 | grep -v "^Sorting\|^Full Text"

# Common useful packages
apt install calf-plugins          # Calf reverb, chorus, EQ, compressor
apt install mda-lv2               # MDA piano, synths, effects
apt install zam-plugins           # ZaMultiCompX2, ZaMaximX2
apt install lsp-plugins           # LSP reverb, parametric EQ, compressor
apt install guitarix              # Guitarix guitar effects as LV2
apt install setBfree              # setBfree organ (if not present)
apt install swh-plugins           # Steve Harris LV2 collection
```

After installing:
```bash
# Regenerate Zynthian's LV2 cache
systemctl restart zynthian
# or via webconf → Engines → Regenerate LV2 Cache
```

### Manual Install (Custom Plugins)

Copy the `.lv2` bundle directory to a path in `LV2_PATH`:

```bash
# Example: copy a downloaded plugin
cp -r ~/MyPlugin.lv2 /zynthian/zynthian-my-data/presets/lv2/

# Verify jalv can find it
jalv --list | grep "MyPlugin"
```

### Via Zynthian Recipes

Zynthian includes install scripts for common plugins in `/zynthian/zynthian-sys/scripts/recipes/`. These handle dependencies and configuration:

```bash
ls /zynthian/zynthian-sys/scripts/recipes/ | grep -i lv2
```

---

## Plugin Presets

LV2 plugins can have their own preset system (stored as `.ttl` files in the bundle). Zynthian exposes these as "banks" in the chain's bank/preset navigation.

Presets are stored in:
- `/zynthian/zynthian-data/presets/lv2/` — factory presets (read-only)
- `/zynthian/zynthian-my-data/presets/lv2/` — user presets

To save a custom preset from the UI:
- Navigate to the plugin's control screen → options → **Save Preset**.

---

## Plugin Controls

LV2 plugin parameters (ports) are automatically mapped to Zynthian controls:

- On V5: encoder 3 navigates parameters, encoder 4 adjusts value
- On touchscreen: tap a parameter in the control grid
- Via MIDI: webconf → MIDI → CC → map CC to a specific parameter

Parameter ranges, defaults, and units come from the plugin's LV2 metadata (`.ttl` file).

---

## Recommended Plugins

| Plugin | Package | Use |
|--------|---------|-----|
| Calf Reverb | `calf-plugins` | High-quality hall/room reverb |
| Calf Compressor | `calf-plugins` | Transparent compressor for dynamics |
| ZaMultiCompX2 | `zam-plugins` | Multiband compressor |
| LSP Parametric EQ | `lsp-plugins` | High-quality parametric EQ |
| MDA Piano | `mda-lv2` | Lightweight electric piano |
| MDA ePiano | `mda-lv2` | Wurlitzer-style electric piano |
| Guitarix (Amp Sim) | `guitarix` | Guitar amp and cabinet simulation |
| setBfree | `setbfree` | Tonewheel organ (see [Synth Engines](synth-engines.md)) |
| Helm | (manual) | Polyphonic FM/subtractive synth [low] |
| Surge XT | (manual) | Advanced hybrid synthesizer [low] |

> **Note:** Helm and Surge XT may require manual compilation on ARM. Performance varies by Pi model. [low]

---

## Troubleshooting LV2

**Plugin not visible in Zynthian after installing:**
```bash
# Check it's in the LV2 path
jalv --list | grep "plugin-name"
# If found, regenerate cache:
systemctl restart zynthian
```

**Plugin crashes / jalv exits immediately:**
```bash
# Test directly
jalv "http://plugin.uri/" 2>&1 | head -20
```

Check if plugin requires specific sample rates or channel counts that don't match current JACK config.

**Plugin loads but produces no sound:**
Verify JACK connections: `jack_lsp -c` shows all ports and connections. The plugin's audio output must be connected to the JACK mixer input.

**Plugin controls don't respond to encoders:**
The plugin may expose many ports. Navigate with encoder 3 until you find the relevant parameter. Parameter names come directly from the plugin's LV2 metadata.

---

## What's Next

- [Synth Engines](synth-engines.md) — all native Zynthian engines including Jalv
- [Audio Setup](audio.md) — JACK configuration
- [Configuration Reference](configuration-reference.md) — LV2_PATH variable

---

*Version: 2026-05-25 — derived from `zynthian-ui/zyngine/zynthian_engine_jalv.py`, `zynthian-sys/config/zynthian_envars_V5.sh`.*
