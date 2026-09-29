---
name: hapax-instrument
description: Use when asked to create a Squarp Hapax instrument definition, or when given a synth, drum machine or effect manual or MIDI implementation chart to turn into a Hapax definition file.
---

# Hapax instrument definition

Turn a manual or MIDI implementation chart into a Squarp Hapax instrument definition (`.txt`) that loads on the first try.
Transcribe exactly; curate with taste; never invent.

## Workflow

### 1. Find the MIDI chart

- **PDF:** search it for "MIDI Implementation", "Control Change", "CC#", "NRPN", "Parameter List" and read only those pages.
  Charts are usually in an appendix.
- **URL:** fetch it; if it is a download page, find and fetch the manual PDF.
- **Images or a scanned chart:** read the images; a table in a picture is still a chart.
- **No MIDI data found:** say so and stop.
  Never fill a definition from general knowledge of the instrument.

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
Use only the directives and sections `format.md` lists: anything else rejects the whole file.

### 7. Validate

If the `hapax` command is available (`command -v hapax`):

    hapax validate --fw <target> <file>

Fix every error and every warning you did not choose, then run it again, until it is clean.
`hapax fix <file>` repairs purely mechanical problems.

Otherwise, go through "Self-check" in `format.md` one item at a time against the file, and fix what fails.

Either way, check Self-check item 4 yourself: `hapax` does not compare names, so two entries that read the same in their first 15 characters pass validation.

### 8. Report

A few lines:

- what was curated (pots, lanes) and why;
- what was left out, and why;
- anything the source left unclear, with the page or table.

If `hapax` was not available, say the file was checked by hand and suggest installing hapax-tui to validate and edit it.
