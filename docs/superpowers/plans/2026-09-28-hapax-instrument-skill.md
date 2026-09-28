# Hapax Instrument Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** A portable Agent Skill, `hapax-instrument`, that turns a manual or MIDI implementation chart into a curated Squarp Hapax instrument definition that loads on the first try.

**Architecture:** One skill folder: `SKILL.md` holds the workflow; `references/format.md`, `references/curation.md` and `references/example.txt` load on demand.
No code: the skill is tested the superpowers:writing-skills way — a baseline run without it (RED), a run with it (GREEN), then fixes for whatever still fails (REFACTOR).
Output is checked with the `hapax` CLI from the sibling `hapax-tui` project.

**Tech Stack:** Markdown (Agent Skills format), the `hapax` CLI (`uv run --project ../hapax-tui hapax`), subagents for test runs.

**Spec:** `docs/superpowers/specs/2026-09-28-hapax-instrument-skill-design.md`

## Global Constraints

- Skill name `hapax-instrument`; folder `hapax-instrument/` at the repo root.
- Portable: no scripts, no dependencies; the skill uses `hapax` only when it is on PATH.
- Default target firmware 3.21; ask the OS only when it changes the file.
- Names: 15 characters shown for entries, 9 for `TRACKNAME`, 27 for the file name before `.txt`.
- Name characters: `A–Z a–z 0–9`, space, `_ - + ! " $ ' ( ) * , . / : < = > ? @`.
- Nothing invented: every CC, NRPN, PC and drum note traces to the source.
- Receive-only: transmit-only messages are never listed.
- Markdown: one sentence per line; files end with a newline; no trailing whitespace.
- Manual PDFs are copyrighted: they live in `eval/manuals/`, which is git-ignored.
- Commits end with `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`.

## Review Focus

1. A MIDI chart that is only an image or scanned table — the skill must read it as images, not report "no MIDI data".
2. More than 64 continuous parameters (the Peak has well over 64) — `[AUTOMATION]` must stop at 64, chosen by `curation.md`, not by manual order.
3. Zero-based program numbers in the manual — `[PC]` must add one.
4. Charts with separate "Transmitted" and "Recognized" columns — only recognized messages are listed.
5. Two long parameter names that share their first 15 characters (`Osc 1 Wave Position` / `Osc 1 Wave Pos Mod`) — shortened names must stay distinct.

Task 5's pass criteria check each of these against the Peak and TR-8S output.

## Conventions for every task

```bash
HAPAX="uv run --project /Users/clayton/work/projects/claytron/hapax-tui hapax"
```

`$HAPAX validate --fw 3.21 FILE` prints findings and a summary line `N files, E errors, W warnings — Hapax OS 3.21`; exit 0 means no errors.

---

### Task 1: Baseline without the skill (RED)

**Files:**
- Create: `.gitignore`
- Create: `eval/prompts.md`
- Create: `eval/baseline/Novation_Peak.txt`, `eval/baseline/TR-8S.txt` (written by subagents)
- Create: `eval/RESULTS.md`
- Not committed: `eval/manuals/*.pdf`

**Interfaces:**
- Produces: `eval/prompts.md` (the two prompts reused verbatim in Task 5), `eval/manuals/peak.pdf`, `eval/manuals/tr-8s-midi.pdf`, and a failure list in `eval/RESULTS.md` under `## Baseline`.

- [ ] **Step 1: Ignore manuals**

`.gitignore`:

```gitignore
eval/manuals/
```

- [ ] **Step 2: Fetch the manuals**

Open `https://downloads.novationmusic.com/novation/synthesisers/peak` and download the latest Peak User Guide PDF (the one with the MIDI parameter chart) to `eval/manuals/peak.pdf`.
Search for Roland's "TR-8S MIDI Implementation" PDF on roland.com and download it to `eval/manuals/tr-8s-midi.pdf`.
If either cannot be downloaded, stop and ask the user for the file.

Run: `ls -la eval/manuals/ && file eval/manuals/*.pdf`
Expected: two files, both `PDF document`.

- [ ] **Step 3: Write the prompts**

`eval/prompts.md`:

````markdown
# Test prompts

Used verbatim for the baseline (without the skill) and the skill run.

## Peak

```text
Create a Squarp Hapax instrument definition for the Novation Peak from the manual at eval/manuals/peak.pdf.
Write it to {OUT}/Novation_Peak.txt.
```

## TR-8S

```text
Create a Squarp Hapax instrument definition for the Roland TR-8S from the MIDI implementation at eval/manuals/tr-8s-midi.pdf.
Write it to {OUT}/TR-8S.txt.
```
````

- [ ] **Step 4: Run the baseline**

Dispatch two `general-purpose` subagents in parallel, one per prompt, with `{OUT}` set to `eval/baseline`.
Add to each prompt only: "Work in /Users/clayton/work/projects/claytron/hapax-skills. Do not read anything under hapax-tui or docs/."
They must not see the skill or the spec.

- [ ] **Step 5: Validate the baseline**

Run: `$HAPAX validate --fw 3.21 eval/baseline/`
Expected: failures. Save the full output.

- [ ] **Step 6: Review by hand**

For each file, check and note every instance of:
- a message not in the manual (spot check 20 entries against the PDF);
- a transmit-only message;
- names over 15 characters, or two names equal in their first 15;
- `CC:n:v` in `[ASSIGN]`, `DEFAULT=NULL`, CC 120–127 in `[ASSIGN]`/`[AUTOMATION]`;
- PC numbers not shifted to 1–128;
- more than 64 `[AUTOMATION]` lines;
- pots on switches or mode selectors;
- anything else surprising, quoted verbatim.

- [ ] **Step 7: Record**

`eval/RESULTS.md`:

```markdown
# Skill test results

## Baseline

### Peak

<validator output, fenced>

- <one line per failure from step 6, quoting the offending line>

### TR-8S

<same>
```

- [ ] **Step 8: Commit**

```bash
git add .gitignore eval/
git commit -m "Record baseline for hapax-instrument skill

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 2: Format reference and example

**Files:**
- Create: `hapax-instrument/references/format.md`
- Create: `hapax-instrument/references/example.txt`

**Interfaces:**
- Produces: `format.md` with sections `## Self-check` (referenced by `SKILL.md` step 7) and `## Firmware differences` (referenced by step 5); `example.txt` (referenced by step 6).

- [ ] **Step 1: Write `example.txt`**

Derived from `hapax-tui/tests/testdata/mine/Matriarch.txt`: same CC numbers, shortened names, curated pots and lanes, no template comments.

```text
VERSION 1
TRACKNAME Matriarch
TYPE POLY
OUTPORT NULL
OUTCHAN NULL
INPORT NULL
INCHAN NULL

[CC]
# Performance
1 Mod Wheel
3 Mod Rate
64 Sustain
65 Glide On
5 Glide Time
85 Glide Type
86 Gated Glide
# Voice
94 Para Mode
95 Multi-Trig
90 Sq LFO Polarity
# Oscillators
74 Osc1 Octave
75 Osc2 Octave
76 Osc3 Octave
77 Osc4 Octave
16 Osc2 Freq
17 Osc3 Freq
18 Osc4 Freq
80 Hard Sync
81 Osc2 Sync
82 Osc3 Sync
83 Osc4 Sync
# Noise
9 Noise Cutoff
# Arpeggiator
73 Arp Play
69 Arp Latch
8 Arp Rate
14 Arp Swing
15 Arp Gate
91 Arp Mode
92 Arp Pattern
93 Arp Range
# Delay
12 Dly Time
13 Dly Spacing
88 Dly Ping Pong
89 Dly Sync
[/CC]

[ASSIGN]
1 CC:16
2 CC:17
3 CC:18
4 CC:5
5 CC:3
6 CC:9
7 CC:12
8 CC:13
[/ASSIGN]

[AUTOMATION]
CC:1
CC:3
CC:5
CC:16
CC:17
CC:18
CC:9
CC:8
CC:14
CC:15
CC:12
CC:13
[/AUTOMATION]

[COMMENT]
Moog Matriarch
Source: Moog Matriarch user manual, MIDI CC chart
[/COMMENT]
```

- [ ] **Step 2: Validate the example**

Run: `$HAPAX validate --fw 3.21 hapax-instrument/references/example.txt && $HAPAX validate --fw 3.10 hapax-instrument/references/example.txt`
Expected: `1 file, 0 errors, 0 warnings` for both. Fix any finding before continuing.

- [ ] **Step 3: Write `format.md`**

```markdown
# Hapax instrument definition format

Source of truth: hapax-tui — its validator design spec and `hapax/rules.py`, verified on Hapax hardware (OS 2.21 and 3.10).
Where this file and hapax-tui disagree, hapax-tui is right.
Squarp's template is not reliable: its name character set is too narrow and some of its limits are not enforced.

## The file

- UTF-8 text, one directive or entry per line.
- `#` starts a comment anywhere on a line.
- Keywords are case-insensitive; numbers may have leading zeros.
- Every directive and every section is optional; leave out empty sections.
- The first mistake rejects the whole file with `SYNTAX ERROR line N`, so a single bad line loses everything.
- File name: the Hapax list shows only the first 27 characters before `.txt`.

## Directives

| Directive | Values |
|---|---|
| `VERSION` | `1` |
| `TRACKNAME` | a name (see Names), or `NULL`; only 9 characters are shown |
| `TYPE` | `POLY`, `DRUM`, `MPE`, `NULL`; `POLYAT`, `AFTR` from 3.20 |
| `OUTPORT` | `A B C D USBD USBH NULL`, `CVGx CVx Gx` (x 1–4), `USBDx USBHx` (x 1–16) from 3.00 |
| `OUTCHAN` | 1–16, `NULL` |
| `INPORT` | `NONE ALLACTIVE A B USBD USBH CVG NULL`, `USBDx USBHx` (x 1–16) from 3.00 |
| `INCHAN` | 1–16, `ALL`, `NULL` |
| `MAXRATE` | `NULL 192 96 64 48 32 24 16 12 8 6 4 3 2 1` |

Any other directive is an error.
`TYPE MPE` with `INPORT A` or `B` loads, but MPE cannot use a DIN input.

## Names

Allowed: `A–Z a–z 0–9`, space, `_ - + ! " $ ' ( ) * , . / : < = > ? @`.
Rejected: ``% & ; [ \ ] ^ ` { | } ~`` and anything non-ASCII (`é`, `°`, `→`, curly quotes, en dashes).
A name is required on every `[CC]`, `[PC]`, `[NRPN]`, `[CC_PAIR]` and `[DRUMLANES]` entry.
Names of any length load, but only the first 15 characters are shown; keep every name within 15 and unique within 15.
`[COMMENT]` lines use the same character set, with no length limit.

## Sections

Each section opens with `[NAME]` and closes with `[/NAME]`.

### `[DRUMLANES]` — `ROW:TRIG:CHAN:NOTE NAME`

- `ROW` 1–16 (1–8 before 3.10); `TRIG` 0–127 or `NULL`; `CHAN` 1–16, `Gx`, `CVx`, `CVGx` (x 1–4) or `NULL`; `NOTE` 0–127 or `NULL`.
- Discarded on a track whose `TYPE` is not `DRUM`.

### `[PC]` — `PC NAME` or `PC:MSB:LSB NAME`

- `PC` 1–128: the file is one-based, the wire is zero-based. A manual listing programs 0–127 means file values 1–128.
- `MSB`, `LSB` 0–127 or `NULL`.

### `[CC]` — `CC NAME` or `CC:DEFAULT=v NAME`

- `CC` 0–127; `DEFAULT` 0–127. `DEFAULT=NULL` is rejected.
- CC 120–127 can be named here but cannot be used in `[ASSIGN]` or `[AUTOMATION]`.

### `[CC_PAIR]` — `MSB_CC:LSB_CC NAME` or `MSB_CC:LSB_CC:DEFAULT=v NAME`

- A 14-bit CC sent as a pair; each CC 0–127, `DEFAULT` 0–16383.

### `[NRPN]` — `MSB:LSB:DEPTH NAME` or `MSB:LSB:DEPTH:DEFAULT=v NAME`

- `MSB` 0–127 or empty; `LSB` 0–127, or 0–16383 only when `MSB` is `0` or empty (`1:200:7` is rejected).
- `DEPTH` 7 or 14; `DEFAULT` 0–127 for 7-bit, 0–16383 for 14-bit.

### `[ASSIGN]` — `POT TARGET` or `POT TARGET DEFAULT=v`

- `POT` 1–8, each at most once.
- `TARGET`: `CC:n` (0–119), `PB`, `AT`, `CV:n` (1–4), `NRPN:MSB:LSB:DEPTH`, `CC_PAIR:MSB:LSB` (from 1.13), `NULL`.
- `DEFAULT`: CC 0–127; NRPN 0–127 or 0–16383 by depth; CC_PAIR 0–16383; CV 0–65535 or `-5V` to `5V`; ignored for PB and AT.
- Write `CC:74 DEFAULT=100`. `CC:74:100` loads, but the default is silently dropped.

### `[AUTOMATION]` — `TARGET` or `TARGET DEFAULT=v`

- Targets and defaults as `[ASSIGN]`, without `NULL`.
- At most 64 entries; the 65th rejects the file.

### `[COMMENT]`

- Free text shown on the Hapax, one or more lines, name character set.

## Firmware differences

Default target: 3.21, the latest.

| Feature | Firmware |
|---|---|
| Section `DEFAULT=` in `[CC]`, `[NRPN]`, `[CC_PAIR]` | applied before 3.00 and from 3.20; **ignored on 3.00–3.10** — for those, put defaults on `[AUTOMATION]` or `[ASSIGN]` lines instead |
| `TYPE POLYAT`, `AFTR` | 3.20 |
| Drum rows 9–16 | 3.10 |
| `USBDx`, `USBHx` ports | 3.00 |
| `CC_PAIR:` in `[ASSIGN]`/`[AUTOMATION]` | 1.13 |
| Tabs | break loading before 1.14; use spaces |

## Self-check

When `hapax` is not installed, check every line of the file against this list before handing it over.

1. Every directive is from the table above, with a listed value.
2. Every section that opens also closes, with the same name.
3. Every name and `[COMMENT]` line uses only allowed characters — look for accents, degree signs, curly quotes, `&`, `%`, `;`, `[`, `]`.
4. Every name is at most 15 characters, and no two names in a section share their first 15.
5. `TRACKNAME` is at most 9 characters, or you accept that it will be cut.
6. `[CC]`: 0–127, each CC once; no `DEFAULT=NULL`.
7. `[PC]`: 1–128.
8. `[NRPN]`: depth 7 or 14; LSB over 127 only with MSB `0` or empty; each address once.
9. `[DRUMLANES]`: only on a `TYPE DRUM` track; rows within the firmware's limit, each once.
10. `[ASSIGN]`: pots 1–8, each once; CC at most 119; defaults written as `DEFAULT=v`, never `CC:n:v`.
11. `[AUTOMATION]`: at most 64 lines; CC at most 119; no `NULL`.
12. Every default is within range for its type and depth, and matches the target firmware (see Firmware differences).
13. The file name is at most 27 characters before `.txt`.
```

- [ ] **Step 4: Check `format.md` against the source**

Compare each table row and rule with `/Users/clayton/work/projects/claytron/hapax-tui/hapax/rules.py` (constants at the top, `_directive`, `_cc`, `_pc`, `_nrpn_address`, `_drum`, `_target`, `_line_default`, `_document`).
Any mismatch: `rules.py` wins; fix `format.md`.

- [ ] **Step 5: Commit**

```bash
git add hapax-instrument/references/
git commit -m "Add format reference and Matriarch example

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 3: Curation reference

**Files:**
- Create: `hapax-instrument/references/curation.md`

**Interfaces:**
- Consumes: the name rules in `format.md`.
- Produces: `curation.md`, loaded by `SKILL.md` step 4.

- [ ] **Step 1: Write `curation.md`**

```markdown
# Curating a definition

The transcription must be complete and exact; the choices below are where taste comes in.
When the manual does not settle a choice, pick the sensible option and say so in the report.

## What to list

- Every message the instrument **receives**. In a chart with "Transmitted" and "Recognized" columns, use "Recognized"; skip rows marked `X` there.
- List a parameter in `[CC]` when it has a CC, and in `[NRPN]` when it has an NRPN; list it in both when it has both.
- Skip channel-mode messages (CC 120–127: All Sound Off, Reset All Controllers, Local Control, All Notes Off, Omni, Mono, Poly) and bank select (CC 0 and 32) unless the instrument uses them for something else.
- Bank select belongs in `[PC]` as `PC:MSB:LSB` when the manual maps banks to patch names.

## Order and grouping

- Keep the manual's order and grouping.
- One comment line per group in `[CC]` and `[NRPN]`: `# Oscillator 1`, `# Filter`, `# Env 2`.

## Names

- At most 15 characters, and no two alike in their first 15.
- Group prefix first, then the parameter: `Osc1 Shape`, `Flt Cutoff`, `Env2 Atk`, `LFO1 Rate`, `FX Mix`.
- Abbreviations: `Osc`, `Flt`, `Env`, `LFO`, `Amp`, `Mod`, `Dly`, `Rev`, `Dist`, `Arp`, `Seq`; `Atk`, `Dec`, `Sus`, `Rel`; `Amt`, `Lvl`, `Freq`, `Reso`, `Fine`, `Crs`, `Pos`, `Depth`, `Rate`, `Sync`, `Mix`.
- Drop words the prefix already implies (`Filter Frequency` in a filter group → `Flt Cutoff`).
- When two names still collide in 15 characters, shorten the part that differs last (`Osc1 WavePos`, `Osc1 WavePosMod`).
- Replace characters the Hapax rejects: `&` → `+` or `and`, `°` → `deg`, accented letters → plain ones.

## Directives

- `TYPE DRUM` when the instrument plays separate voices on separate notes (drum machines, samplers in kit mode); `POLY` otherwise.
- `MPE`, `POLYAT` or `AFTR` only when the manual documents MPE or polyphonic aftertouch.
- `OUTPORT`, `OUTCHAN`, `INPORT`, `INCHAN` are `NULL` unless the user says how the instrument is connected: `NULL` keeps whatever the track already has.
- `TRACKNAME`: the instrument's name, 9 characters or fewer if possible (`Peak`, `TR-8S`, `Matriarch`).

## Drum lanes

- `ROW:NULL:NULL:NOTE NAME`: `NULL` channel follows the track's channel, `NULL` trig uses the Hapax default.
- Rows in the instrument's panel order, first voice on row 1.
- More than 8 voices: rows 9–16 need firmware 3.10; ask if the target is older.
- When one voice responds to several notes, give it the note the manual lists first.

## Pots — `[ASSIGN]`, all 8

What a player reaches for live, in this order of preference:

1. Filter cutoff and resonance.
2. Filter envelope amount.
3. The main timbre control (wave shape, wavetable position, FM amount).
4. One or two LFO or modulation controls (rate, depth).
5. An effect mix or time (delay, reverb).
6. Amp envelope decay or release.

For a drum machine: per-voice level, tune or decay for the main voices, or global effects.
Never a switch, mode or on/off control, and never a patch-global setting that jumps the sound (octave, voice mode, arp on).
For the same parameter, prefer the NRPN or CC pair when it has more resolution than the CC.

## Automation lanes — `[AUTOMATION]`, at most 64

- Continuous parameters that shape a sound over time, in the same grouping as `[CC]`.
- Include every pot's target.
- Switches only when they are musically useful to sequence (arp on, sync on).
- More than 64 candidates: drop the least expressive (fine tune, setup parameters, global settings) first; say what was dropped in the report.
- Prefer the higher-resolution message, as for pots.

## Program changes

- List patch names only when the manual gives them; otherwise leave `[PC]` out.
- Convert zero-based numbering: manual program 0 is file PC 1.

## Defaults

- Only a resting value the manual states: pan centre at 64, a documented init value.
- Never invent one.
- On 3.00–3.10, section defaults are ignored; put the default on the `[AUTOMATION]` line instead.

## `[COMMENT]`

- Line 1: the instrument's name.
- Line 2: `Source: ` and the manual's title and version, as given on its cover.
```

- [ ] **Step 2: Check names in the example obey the rules**

Run: `grep -E '^[0-9]+ ' hapax-instrument/references/example.txt | cut -d' ' -f2- | awk 'length > 15'`
Expected: no output.

- [ ] **Step 3: Commit**

```bash
git add hapax-instrument/references/curation.md
git commit -m "Add curation reference

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 4: `SKILL.md` and README

**Files:**
- Create: `hapax-instrument/SKILL.md`
- Create: `README.md`

**Interfaces:**
- Consumes: `references/format.md` (sections `## Self-check`, `## Firmware differences`), `references/curation.md`, `references/example.txt`.

- [ ] **Step 1: Write `SKILL.md`**

```markdown
---
name: hapax-instrument
description: Use when asked to create a Squarp Hapax instrument definition, or when given a synth, drum machine or effect manual or MIDI implementation chart to turn into a Hapax definition file.
---

# Hapax instrument definition

Turn a manual or MIDI implementation chart into a Squarp Hapax instrument definition (`.txt`) that loads on the first try.
Transcribe exactly; curate with taste; never invent.

## Workflow

### 1. Find the MIDI chart

- **PDF:** search it for "MIDI Implementation", "Control Change", "CC#", "NRPN", "Parameter List" and read only those pages. Charts are usually in an appendix.
- **URL:** fetch it; if it is a download page, find and fetch the manual PDF.
- **Images or a scanned chart:** read the images; a table in a picture is still a chart.
- **No MIDI data found:** say so and stop. Never fill a definition from general knowledge of the instrument.

### 2. Extract

List every message the instrument **receives**: CCs, NRPNs (MSB, LSB, 7- or 14-bit), 14-bit CC pairs, program change and bank select, and the drum note map.
Skip rows the chart marks as transmitted only.
Note where each came from (page or table) for the report.

### 3. Load the rules

Read `references/format.md` and `references/curation.md`, and look at `references/example.txt` for the target style.

### 4. Curate

Follow `references/curation.md`: directives, names, grouping, the 8 pots, at most 64 automation lanes.

### 5. Ask only what changes the file

The target firmware is 3.21 unless the user says otherwise.
Ask the user's Hapax OS version only when the file would differ (see "Firmware differences" in `format.md`): section defaults, drum rows 9–16, `POLYAT`/`AFTR`.
Ask drum or poly only when the instrument is genuinely both.
One question at a time, with the answer you recommend.

### 6. Write

Save `<Instrument>.txt`, at most 27 characters before `.txt`, in the style of `references/example.txt`: directives, then sections with one comment line per group, no template boilerplate, empty sections left out.

### 7. Validate

If the `hapax` command is available (`command -v hapax`):

    hapax validate --fw <target> <file>

Fix every error and every warning you did not choose, then run it again, until it is clean.
`hapax fix <file>` repairs purely mechanical problems.

Otherwise, go through "Self-check" in `format.md` one item at a time against the file, and fix what fails.

### 8. Report

A few lines:

- what was curated (pots, lanes) and why;
- what was left out, and why;
- anything the source left unclear, with the page or table.

If `hapax` was not available, say the file was checked by hand and suggest installing hapax-tui to validate and edit it.
```

- [ ] **Step 2: Write `README.md`**

```markdown
# hapax-skills

AI skills for the Squarp Hapax.

## hapax-instrument

Point it at a manual or MIDI implementation chart and it writes a Hapax instrument definition: every CC, NRPN, program change and drum note the instrument receives, with short names, 8 pots and a set of automation lanes chosen for you.
It targets Hapax OS 3.21 unless you tell it otherwise.

### Install

- **Claude Code:** copy or symlink `hapax-instrument/` into `~/.claude/skills/`.
- **claude.ai:** zip the `hapax-instrument/` folder and upload it under Settings → Capabilities → Skills.
- **Other Agent Skills hosts:** install the `hapax-instrument/` folder as a skill.

### Use

> Make a Hapax instrument definition for my Novation Peak from this manual: peak-user-guide.pdf

### Validating

The skill checks its output by hand unless the `hapax` command from [hapax-tui](https://github.com/claytron/hapax-tui) is installed, in which case it validates against the real rules and fixes what it finds.
hapax-tui also edits definitions in the terminal.
```

- [ ] **Step 3: Check the frontmatter and references**

Run: `head -4 hapax-instrument/SKILL.md && ls hapax-instrument/references/`
Expected: `name: hapax-instrument` and a `description:` starting "Use when"; `curation.md example.txt format.md`.

Run: `grep -o 'references/[a-z.]*' hapax-instrument/SKILL.md | sort -u | while read f; do test -f hapax-instrument/$f && echo "ok $f" || echo "MISSING $f"; done`
Expected: three `ok` lines.

- [ ] **Step 4: Commit**

```bash
git add hapax-instrument/SKILL.md README.md
git commit -m "Add hapax-instrument skill and README

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---

### Task 5: Test with the skill (GREEN, then REFACTOR)

**Files:**
- Create: `eval/skill/Novation_Peak.txt`, `eval/skill/TR-8S.txt` (written by subagents)
- Modify: `eval/RESULTS.md`
- Modify as needed: `hapax-instrument/SKILL.md`, `hapax-instrument/references/*.md`

**Interfaces:**
- Consumes: `eval/prompts.md` and the manuals from Task 1; the skill from Tasks 2–4.

- [ ] **Step 1: Run with the skill**

Dispatch two `general-purpose` subagents in parallel, each with its prompt from `eval/prompts.md`, `{OUT}` set to `eval/skill`, prefixed by:

- Peak: "Read /Users/clayton/work/projects/claytron/hapax-skills/hapax-instrument/SKILL.md and follow it. The `hapax` command is available as `uv run --project /Users/clayton/work/projects/claytron/hapax-tui hapax`. Target firmware 3.21. Do not read anything under hapax-tui other than running that command, or under docs/ or eval/baseline/."
- TR-8S: "Read /Users/clayton/work/projects/claytron/hapax-skills/hapax-instrument/SKILL.md and follow it. The `hapax` command is not installed; do not look for it. Target firmware 3.21. Do not read anything under hapax-tui, docs/ or eval/baseline/."

The TR-8S run exercises the self-check path; the Peak run exercises the CLI path.

- [ ] **Step 2: Validate**

Run: `$HAPAX validate --fw 3.21 eval/skill/ && $HAPAX validate --fw 3.10 eval/skill/`
Expected: `0 errors` both times. Warnings on 3.10 are acceptable only if they are section defaults (`default.ignored`).

- [ ] **Step 3: Review against pass criteria**

For each file:

- Spot check 20 entries against the PDF: every one appears in the manual as a received message.
- No names over 15 characters; no two equal in their first 15 (Review Focus 5):
  `grep -vE '^\s*(#|\[|$)' FILE | sed -E 's/^[^ ]+ //' | cut -c1-15 | sort | uniq -d` prints nothing.
- Peak: `[AUTOMATION]` has at most 64 lines, and the dropped parameters are the least expressive, named in the report (Review Focus 2).
- Peak: every pot target is continuous, per `curation.md`.
- TR-8S: `TYPE DRUM`, `[DRUMLANES]` rows in panel order, notes matching the chart; recognized-only messages (Review Focus 4).
- Any `[PC]` entry is one-based (Review Focus 3).
- The subagent's report names ambiguities and, for TR-8S, says the file was checked by hand.

Review Focus 1 (image-only chart): take a screenshot of one page of the Peak MIDI chart to `eval/manuals/peak-chart.png`, dispatch a third subagent with SKILL.md and "Create a Hapax definition for the Novation Peak from eval/manuals/peak-chart.png; write it to eval/skill/Peak_Image.txt", and check it reads the chart rather than reporting no MIDI data.

- [ ] **Step 4: Record**

Append to `eval/RESULTS.md`:

```markdown
## With the skill

### Peak

<validator output, fenced>

- <each baseline failure: fixed / still present>
- <each pass criterion: pass / fail, with the offending line>

### TR-8S

<same>

### Image-only chart

<pass / fail, one line>
```

- [ ] **Step 5: REFACTOR — close the gaps**

For every failure still present: find the sentence in `SKILL.md`, `format.md` or `curation.md` that should have prevented it, and make it explicit (a rule, a red flag, or an example).
Rerun only the failing subagent (Step 1) and Steps 2–4.
Repeat until every criterion passes.
Stop and ask the user if a failure survives two rounds.

- [ ] **Step 6: Commit**

```bash
git add eval/ hapax-instrument/
git commit -m "Test hapax-instrument skill on Peak and TR-8S

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```
