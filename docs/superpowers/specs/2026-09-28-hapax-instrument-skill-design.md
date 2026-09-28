# Hapax Instrument Skill — Design

- Date: 2026-09-28
- Status: draft, under review

## Context

Squarp Hapax instrument definitions are `.txt` files that name an instrument's CCs, NRPNs and program changes, map drum lanes, and preassign pots and automation lanes.
Writing one means reading a manual's MIDI implementation chart, transcribing it, and making dozens of small choices within a format the Hapax enforces strictly and reports poorly.

`hapax-tui` (sibling project) validates, fixes and edits definitions.
Its rules were verified on hardware and are the source of truth for the format.
This project adds an AI skill that writes the first draft: point it at a manual or MIDI chart and get a curated definition that loads on the first try.

## Goals

- Input: a PDF manual, a URL, pasted text, or images of a MIDI chart.
- Output: one `<Instrument>.txt` definition, transcribed faithfully and curated with opinions (pots, automation lanes, short names).
- Portable: a standard Agent Skill usable in Claude Code, claude.ai and other Agent Skills hosts, with no install beyond the skill folder.
- Uses the `hapax` CLI to validate when it is on PATH; works without it.
- Targets firmware 3.21 by default; asks the user's OS only when the answer changes the output.

### Success

- The file validates cleanly with `hapax validate` on 3.21 (and on 3.10 when asked to target it).
- Every CC, NRPN, PC and drum note traces to a line of the source; nothing is invented.
- Only incoming messages are listed, never transmit-only ones.
- The user's remaining edits are matters of taste, not correctness.

### Non-goals

- No bundled validator: `hapax-tui` is the validator, recommended in the README once it is public.
- No intermediate JSON or second skill.
- No editing of existing definitions; this skill writes new ones.
- No fetching of community definitions.

## Layout

```text
hapax-skills/
  README.md                      what it is, how to install the skill, hapax-tui recommendation
  hapax-instrument/              the skill; copy or symlink this folder to install
    SKILL.md
    references/
      format.md
      curation.md
      example.txt
```

## `SKILL.md`

Frontmatter `name: hapax-instrument`.
The description triggers on requests to create a Squarp Hapax instrument definition, or on a MIDI implementation chart or manual supplied with a mention of the Hapax.

The body is the workflow, each step short:

1. **Find the chart.**
   A large PDF: search it for "MIDI Implementation", "Control Change", "CC#", "NRPN" and read only those pages.
   A URL: fetch it; follow a link to the manual PDF if the page is a download index.
   Images: read them.
   No MIDI data found: say so and stop.
2. **Extract** every message the instrument receives: CC, NRPN (MSB, LSB, 7- or 14-bit), 14-bit CC pairs, program change and bank select, drum note map.
   Skip transmit-only rows.
   Keep a note of where each came from (page or table).
3. **Directives.**
   `TYPE DRUM` only for an instrument with per-voice notes, `POLY` otherwise; `MPE`, `POLYAT` or `AFTR` only when the manual supports them.
   `OUTPORT`, `OUTCHAN`, `INPORT`, `INCHAN` are `NULL` unless the user says how the instrument is wired.
   `TRACKNAME` is the instrument's short name.
4. **Curate** with `references/curation.md`.
5. **Ask** only when the answer changes the file: the target OS when section defaults, drum rows 9–16 or `POLYAT`/`AFTR` are involved; drum versus poly when the instrument is genuinely both.
   One question at a time, with a recommended answer.
6. **Write** `<Instrument>.txt`, at most 27 characters before `.txt`, in the lean style of `references/example.txt`.
7. **Validate.**
   If `hapax` is on PATH: `hapax validate --fw <target> <file>`, fix, repeat until clean; `hapax fix` may be used for mechanical repairs.
   Otherwise: go through the self-check list in `references/format.md`, line by line.
8. **Report** in a few lines: what was curated and why, what was left out, what the source left ambiguous.

`SKILL.md` loads `format.md` before writing and `curation.md` before curating, so neither sits in context until needed.

## `references/format.md`

A condensed restatement of `hapax-tui`'s rules (its validator design spec and `rules.py`), headed by a line naming them as the source so drift can be checked.
It contains shapes and rules only, not the evidence behind them.

- Every directive with its allowed values.
- Every section's entry shape: `[DRUMLANES]`, `[PC]`, `[CC]`, `[CC_PAIR]`, `[NRPN]`, `[ASSIGN]`, `[AUTOMATION]`, `[COMMENT]`.
- The name character set as the hardware accepts it: `A–Z a–z 0–9`, space, `_ - + ! " $ ' ( ) * , . / : < = > ? @`; `[COMMENT]` uses the same set.
- Display limits: 15 characters for entry names, 9 for `TRACKNAME`, 27 for file names.
- Rules that catch people out:
  - CC 120–127 can be named but not used in `[ASSIGN]` or `[AUTOMATION]`.
  - `DEFAULT=NULL` is rejected.
  - In `[ASSIGN]`, `CC:74:100` loads but drops the default; write `CC:74 DEFAULT=100`.
  - NRPN LSB may exceed 127 only when MSB is `0` or empty.
  - At most 64 `[AUTOMATION]` entries.
  - PC is 1–128 in the file (0–127 on the wire): add one to a manual's zero-based program numbers.
  - `[DRUMLANES]` is discarded on a non-DRUM track.
- The firmware-dependent rules table for 1.12–3.21, including section `DEFAULT=` being ignored on 3.00–3.10 (put defaults on `[AUTOMATION]` lines for those).
- The self-check list used when `hapax` is not installed.

## `references/curation.md`

- **Pots (8):** what a player reaches for live — filter cutoff and resonance, filter envelope amount, main timbre or oscillator shape, one or two LFO controls, FX mix.
  Never a mode switch, on/off toggle, or patch-global setting that jumps the sound.
  Prefer an NRPN over a CC for the same parameter when it has more resolution.
- **Automation (≤ 64):** continuous parameters first, in the same grouping as `[CC]`; switches only when musically useful.
- **Names:** consistent prefixes per group (`Osc1`, `Flt`, `Env2`, `LFO1`, `FX`), common abbreviations (`Cutoff`, `Reso`, `Atk`, `Dec`, `Sus`, `Rel`, `Amt`, `Lvl`), each unique within its first 15 characters.
- **CC and NRPN for one parameter:** list both where the instrument accepts both; pots and lanes use the higher-resolution one.
- **Drums:** `ROW:NULL:NULL:NOTE NAME`, so lanes follow the track's channel; rows in the instrument's panel order.
- **Program changes:** list factory patch names only when the manual gives them; otherwise leave `[PC]` empty.
- **Defaults:** only a resting value the manual states (pan centre, a documented init value); never invented.
- **Grouping:** a one-line comment per group in `[CC]` and `[NRPN]` (`# Filter`), in manual order.
- **`[COMMENT]`:** instrument name and the source (manual title and version).

## `references/example.txt`

A lean, curated definition for the Moog Matriarch, derived from the author's `Matriarch.txt` in `hapax-tui/tests/testdata/mine/`.
It shows the target style: directives, grouped sections with one-line comments, curated `[ASSIGN]` and `[AUTOMATION]`, source in `[COMMENT]`, no template boilerplate.
It must validate cleanly on 3.21.
Peak and TR-8S are held out as test cases, so the example is not one of them.

## `README.md`

What the skill does, how to install it (copy or symlink `hapax-instrument/` into `~/.claude/skills/`, or upload the folder as a skill on claude.ai), and a pointer to `hapax-tui` for validation and editing once it is public.

## Testing

Following superpowers:writing-skills: baseline first, then the skill, on the same inputs.

**Inputs:**

- Novation Peak, from `https://downloads.novationmusic.com/novation/synthesisers/peak` — poly synth, CCs and NRPNs.
- Roland TR-8S public manual or MIDI implementation — drum machine, drum note map.

The author's hand-written `Novation_Peak.txt` and `TR-8S.txt` are references for comparison, not answers the output must match.

**Baseline:** a subagent without the skill writes both definitions.
Record every failure: invented or transmit-only messages, rejected characters, names over 15 characters, `CC:74:100` in `[ASSIGN]`, wrong PC numbering, and whatever else appears.

**With the skill:** the same prompts, the skill loaded.
Pass when:

- `hapax validate --fw 3.21` reports no errors on either file; warnings are reviewed and justified.
- A spot check of 20 entries per file traces each to the source.
- Pots and lanes are defensible against `curation.md`.

Each baseline failure the skill does not fix becomes an edit to `SKILL.md` or a reference file, and the test is rerun.

## Decisions worth recording

**Standalone rather than dependent on hapax-tui.**
The skill is meant to be shared, and most Hapax owners will not have the CLI.
The cost is a restatement of the rules in `format.md` that can drift; its header names the source so drift is easy to check.

**No bundled validator.**
Copying `rules.py` and `grammar.lark` would guarantee a real check everywhere, but doubles the rules to maintain and needs code execution with Lark installed.
The CLI already exists; the skill uses it when present.

**Curated, not a bare transcription.**
A bare list is the easy part; choosing 8 pots and naming 100 parameters in 15 characters is the tedium worth delegating.
The user edits taste; the skill must not invent.

**Firmware 3.21 by default.**
It matches the validator's default and what most owners run; the skill asks only when the target changes the file.
