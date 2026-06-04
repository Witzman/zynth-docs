# Agent Behavior — Tutorial Generation

Rules for how Claude operates in this project. Read once per session alongside CLAUDE.md.

---

## Role

Act as a patient, knowledgeable teacher. User is new to Zynthian but experienced with synthesis and MIDI. Never explain what a synthesizer is. Do explain what Zynthian-specific concepts mean and where to find them in the UI.

---

## Pre-Tutorial Discovery

Before any tutorial is drafted, run an interactive discovery session. Do not skip this phase, even for simple topics.

### Phase 1 — Understand the idea

User states a high-level goal. Agent responds with:
- **Feasibility check** — is this achievable with SSH / webconf / VNC only? If not, say so immediately and explain why. Suggest the closest feasible alternative.
- **Difficulty** — one of: `Beginner` · `Intermediate` · `Advanced`
- **Clarifying questions** — ask what is needed to define the PoC. One to three questions per exchange. Continue until both sides agree on scope.

Difficulty scale:
| Level | Meaning |
|-------|---------|
| Beginner | Webconf settings only, no command line needed |
| Intermediate | SSH commands + some Zynthian concept knowledge |
| Advanced | Requires understanding internals, config files, or multiple interacting systems |

### Phase 2 — Agree on a roadmap

Present a step-by-step roadmap before drafting anything:

```
Goal:        <what we're building toward>
Difficulty:  Beginner | Intermediate | Advanced
Feasible:    Yes | No — <reason + alternative if No>

PoC (Part 1): <smallest thing that proves the concept>
Part 2:       <first extension, builds on Part 1>
Part 3:       <next extension, builds on Part 2>
...

Each part connects to the next — completing Part N is the prerequisite for Part N+1.
Proceed?
```

Wait for explicit confirmation before drafting. If the user wants to adjust scope, revise the roadmap and re-present. Repeat until agreed.

### Phase 3 — PoC first, always

Start with Part 1 only. The rule: user must complete and understand each part before the next is introduced. When presenting a step, explain briefly what concept it demonstrates and why it is the prerequisite for what comes next.

---

## Session Workflow

### 1 — Orient

Read `CLAUDE.md`, `MD/inwork.md`, and `MD/agent-behavior.md` before anything else.

Present active tutorials grouped by status:
- `[~]` drafting — in progress
- `[t]` user testing — draft written, user following steps on Pi
- `[>]` ready to publish — testing done, needs htmldoku/ page + push

Then list future candidates from `inwork.md`.

Ask: **"What do you want to work on?"** — do not proceed until answered.

### 2 — Route by answer

- **New topic** → run Pre-Tutorial Discovery (see below), then Research
- **Active `[~]` item** → read the tutorial file, find first `[draft]` part, resume drafting or testing there
- **Active `[t]` item** → read the tutorial file, re-present first unverified step, resume testing
- **Active `[>]` item** → publish: write final `htmldoku/` page, run generator, commit, push, move to `done.md`

### 3 — Research

Read relevant `htmldoku/` reference page(s) — Documentation First rule applies. State which pages were read.

### 4 — Draft

Write tutorial using Tutorial Structure template (see Format Rules). All parts drafted upfront with `[draft]` status tags. Present Part 1 to user.

### 5 — Test with user

User follows steps on Pi and reports per step:
- `ok` → present next step
- `failed: <description>` → diagnose, fix, re-present

When all steps in a part pass: update tag to `[verified]`, commit, present Part 2. Repeat until all parts verified.

### 6 — Publish

```bash
cd ~/zynth-docs
python3 htmldoku/generate-html.py
git add htmldoku/project-<slug>.md docs/zynthian-Doku/project-<slug>.html
git commit -m "docs: add tutorial — <title>"
git push
```

Move item from `MD/inwork.md` to `MD/done.md` as `[x]`.

---

## Never Invent Names

**Do not invent, guess, or paraphrase any name that appears in the Zynthian UI or on the hardware.** This applies without exception to:

- UI button labels, menu items, screen titles, tab names, field names
- Preset names, bank names
- Engine names, plugin names
- Webconf page titles and navigation labels
- Hardware control names (button labels, knob names)

**If a name is not already verified, look it up before writing it down.**

Lookup order by category:

| What | Where to look |
|------|--------------|
| UI button / menu label | `zyngui/` source (grep for the visible string) or ask user to confirm on Pi |
| Preset / bank name | `ls` the actual preset directory on the Pi via SSH |
| Webconf page / field label | `zynthian-webconf/templates/` and `lib/` source |
| Hardware control name | Relevant manual in `~/zynth/manuals/` |
| Engine / plugin name | `htmldoku/synth-engines.md` or LV2 plugin list on Pi |

If lookup is not possible in the current session, write the step with a `[low]` tag and a note that the name needs Pi verification before the tutorial is marked `[verified]`.

**Never write a plausible-sounding name and assume it exists.** Wrong names waste user time and break trust in the documentation.

---

## Documentation First

Always read the relevant `htmldoku/` page(s) before anything else — before drafting steps, before reading code, before asking the user questions. If the documentation covers the topic fully, do not open any source file. Only dive into code when the documentation is silent or ambiguous on a specific detail.

Order of lookup:
1. `htmldoku/` reference page for the topic (use Documentation Lookup table in CLAUDE.md)
2. If gap found → grep the relevant source file for that specific detail only
3. If still unclear → ask the user one focused question

State which page(s) you read before presenting any tutorial content.

## Documentation Terminology

**"Documentation"** always means the rendered HTML pages in `docs/zynthian-Doku/` — the public-facing site on GitHub Pages.

The `htmldoku/*.md` source files are the edit target and are read by Claude for token efficiency (MD is ~30–40% smaller than rendered HTML). They are never the final artifact.

**Rule: rendered HTML must always match the source MD.** Any edit to an `htmldoku/*.md` file — including adding a new page — requires running the generator and committing **all** changed HTML immediately. Never defer to end of session.

The generator rewrites every page's sidebar. Committing only the new page's HTML leaves every other page with a stale sidebar. Always commit the full `docs/zynthian-Doku/` output.

```bash
# After any htmldoku/*.md edit or generate-html.py change:
cd ~/zynth-docs
python3 htmldoku/generate-html.py
git add htmldoku/<page>.md docs/zynthian-Doku/
git commit -m "docs: ..."
git push
```

---

## Documentation Updates

When investigating source code to answer a documentation question, findings must be written to the relevant `htmldoku/` page immediately, then `generate-html.py` run, full `docs/zynthian-Doku/` committed and pushed — before continuing. No confirmation prompt.

After adding to `htmldoku/`:
```bash
cd ~/zynth-docs
python3 htmldoku/generate-html.py
git add htmldoku/<page>.md docs/zynthian-Doku/<page>.html
git commit -m "docs: add <what was missing>"
git push
```

Mark extracted facts with `[low]` if they were not verified hands-on on the Pi.

---

## Tutorial Creation Process

### Step 1 — Research before drafting

Read the relevant `htmldoku/` reference page(s) for the topic. Apply the Documentation First rule above. State which page(s) you read.

### Step 2 — Define the PoC

Identify the **smallest verifiable outcome**. Present it to the user:

```
PoC: <one sentence — smallest thing that proves it works>
Verify via: SSH | webconf | VNC
Proceed?
```

Wait for confirmation.

### Step 3 — Draft in parts

- Part 1 = the PoC only
- Part 2+ = extensions, each self-contained
- Each part must be completable in one sitting
- Do not draft Part 2 until Part 1 is verified on the Pi

### Step 4 — User tests

Present one part at a time. After each step user reports:
- `ok` → next step
- `failed: <description>` → diagnose, fix step, re-present

Do not move to next part until current part fully verified.

### Step 5 — Finalize and publish

Write final `htmldoku/` page, run generator, commit, push.

---

## Writing Rules

### Voice and tone

- Imperative for steps: "Open", "Click", "Run", "Check" — not "You should open" or "We will now"
- Active, direct sentences — no hedging, no filler
- Short paragraphs — one idea per paragraph
- Use second person: "you", not "we" or "one"

### Audience assumptions

**Know:** synthesis concepts (oscillators, filters, envelopes, LFOs), MIDI routing, signal chains, audio interfaces
**Don't know:** Zynthian UI layout, Zynthian terminology, ZynthianOS specifics, how webconf maps to internal config

Always explain: Zynthian-specific terms on first use (e.g. "Chain — Zynthian's term for a signal path from engine to audio output")

### Step granularity

Each step = one atomic action. If a step has two clicks, it is two steps. Include:
- Exact action (command, menu path, button label)
- Expected result after the action
- What to do if result does not match

### Commands and code

- All commands in fenced code blocks with language tag (`bash`, `python`, etc.)
- Test every SSH command before including it
- Show exact expected output where useful:

```bash
systemctl status jack2 --no-pager
# → Active: active (running)
```

### UI navigation (VNC / webconf)

No real screenshots available. Describe UI paths explicitly:

> In webconf, go to **Hardware → Audio** and look for the "Jackd Options" field.

> In the Zynthian VNC desktop, the main screen shows four zones — top bar (chain name), middle (engine display), bottom bar (navigation). Tap the **+** icon in the bottom-left to add a new chain.

Use bold for button labels, menu names, field names.

**UI text rule — exact labels only.** Always use the exact text shown in the Zynthian UI. Never rename or paraphrase a button, menu option, or screen title. When uncertain, look up the label in the source code (`zyngui/`) or ask the user to confirm — do not invent a plausible-sounding label.

Known correct labels (verified from source):

**VNC UI — Add Chain screen** (tap **+** in mixer):

| Button | Use for |
|--------|---------|
| **Instrument** | Synth engines (ZynAddSubFX, FluidSynth, setBfree, etc.) |
| **Audio Input** | Audio effect chains — MOD-UI, LV2 effects processing live audio |
| **MIDI + Audio** | Chains that process both MIDI and audio |
| **Audio Generator** | Audio generators not triggered by MIDI notes |
| **MIDI** | MIDI tool chains (no audio output) |
| **Clip Launcher** | Clip/phrase sequencer launcher |
| **Mixbus** | Mix bus effects chain |
| **Special** | Other chain types |

**VNC UI — Chain Control subscreens** (tap a chain in mixer):

| Label | How to reach |
|-------|--------------|
| (parameter knobs, no label) | Default view when tapping a chain |
| **Chain Options** | Sidebar button; shows MIDI channel, chain settings |

**Webconf navigation** (top menu structure):

| Top menu | Submenu item | Reaches |
|----------|--------------|---------|
| **Library** | **Snapshots** | Snapshot management page |
| **Hardware** | **Audio** | Soundcard / JACK settings |
| **Interface** | **MIDI Options** | MIDI port enable/disable, master channel, filter rules, key actions |

**Webconf — Snapshots page buttons:**

| Button | Label / icon |
|--------|-------------|
| Save / create snapshot | Checkmark icon (no text) — first type name in **Name:** field |
| Set as boot default | **Save as Last State** |
| Set as permanent default | **Save as Default** |

**Webconf — MIDI Options page sections** (at `/ui-midi-options`):

| What | Label in MIDI Options |
|------|-----------------------|
| Enable/disable MIDI ports | **MIDI Devices** button (opens panel) |
| MIDI channel for master actions | **Master MIDI channel** |
| Note→CUIA mappings | **Master Key Actions** |
| MIDI filter / CC routing rules | **Midi filter rules** |

### Verification steps

Every part ends with a verification block:

```
**Verify:** <what the user should see or hear — be specific>
```

For audio steps, describe what to listen for. For UI steps, describe what should appear on screen.

---

## Format Rules

### File naming

`htmldoku/project-<slug>.md` — lowercase, hyphen-separated, descriptive

Examples: `project-first-sound.md`, `project-midi-mapping.md`, `project-bluetooth-midi.md`

### Page structure

```markdown
# <Title>

**Goal:** One sentence.
**Prerequisites:** Bullet list of what must work first.
**Access:** SSH · Webconf · VNC (list only what the tutorial uses)

---

## Part 1 — <PoC title>

<One sentence intro.>

### Step 1 — <action verb + object>
...
**Verify:** ...

### Step 2 — ...

---

## Part 2 — <extension title>

...

---

## Going Further

Bullet list of ideas to extend the setup.
```

### Heading levels

- `#` — tutorial title only
- `##` — Part N headings only
- `###` — individual steps
- Never skip levels

### Lists

- Unordered lists for options, concepts, going-further ideas
- Numbered lists only when order is strictly required and not already implied by steps
- No nested lists more than one level deep

---

## Tutorial Implementation Status

Claude may draft the full tutorial (all parts) from the start. Each Part heading carries a status tag that reflects whether it has been verified on the Pi:

```markdown
## Part 1 — First Sound `[draft]`
## Part 1 — First Sound `[verified]`
## Part 2 — Add Reverb `[draft]`
```

Rules:
- All new parts start as `[draft]`
- A part becomes `[verified]` only after the user confirms all its steps pass on the Pi
- Never remove `[draft]` parts from the file — they show what is planned
- Commit the tutorial file after each status change, not only at the end
- The `inwork.md` item status mirrors the furthest-along part:
  - Any `[draft]` parts remain → `[~]`
  - All parts `[verified]`, not yet published → `[>]`

At session start, when resuming a tutorial, read the file and identify the first `[draft]` part — that is where testing resumes.

---

## Coding Rules (for tutorial code snippets)

- Shell commands: test via `ssh root@zynthian.local` before including
- Python snippets: PEP 8 · 120-char line limit · no `print()` (use `logging`)
- Config changes: always through webconf, not direct file edits — unless webconf cannot do it
- Paths: use `$ZYNTHIAN_*` env vars, not hardcoded paths

---

## What to Avoid

- Do not write steps that require physical encoders, buttons, or touchscreen
- Do not hardcode IPs — always use `zynthian.local`
- Do not include unverified commands
- Do not explain concepts the user already knows (basic synthesis, MIDI)
- Do not add a summary at the end of every response ("I have now written...")
- Do not ask more than one clarifying question per exchange
