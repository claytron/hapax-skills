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
`#` starts a comment even inside a name: `Tom #2` loads silently as `Tom`, and `#2 Tom` leaves no name and rejects the file.
Never use `#` in a name or `[COMMENT]` line.
A name is required on every `[CC]`, `[PC]`, `[NRPN]`, `[CC_PAIR]` and `[DRUMLANES]` entry.
Names of any length load, but only the first 15 characters are shown; keep every name within 15 and unique within 15.
`[COMMENT]` lines use the same character set, with no length limit.

## Sections

Each section opens with `[NAME]` and closes with `[/NAME]`.
Only the sections below exist; any other section name rejects the file at its first entry.

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
| Section `DEFAULT=` in `[CC]`, `[NRPN]`, `[CC_PAIR]` | applied before 3.00 (`[CC_PAIR]` from 1.14) and from 3.20; **ignored on 3.00–3.10** — for those, put the default on the `[AUTOMATION]` line instead, the only default they apply |
| `TYPE POLYAT`, `AFTR` | 3.20 |
| Drum rows 9–16 | 3.10 |
| `USBDx`, `USBHx` ports | 3.00 |
| `CC_PAIR:` in `[ASSIGN]`/`[AUTOMATION]` | 1.13 |
| Tabs | break loading before 1.14; use spaces |

## Self-check

When `hapax` is not installed, check every line of the file against this list before handing it over.

1. Every directive is from the table above, with a listed value.
2. Every section that opens also closes, with the same name.
3. Every name and `[COMMENT]` line uses only allowed characters — look for accents, degree signs, curly quotes, `#`, `&`, `%`, `;`, `[`, `]`.
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
