# Curating a definition

The transcription must be complete and exact; the choices below are where taste comes in.
When the manual does not settle a choice, pick the sensible option and say so in the report.

## What to list

- Every message the instrument **receives**.
  In a chart with "Transmitted" and "Recognized" columns, use "Recognized"; skip rows marked `X` there.
- List a parameter in `[CC]` when it has a CC, and in `[NRPN]` when it has an NRPN; list it in both when it has both.
- A 14-bit CC pair ("CC pair 29,61", or MSB on CC n and LSB on CC n+32) goes in `[CC_PAIR]` as `29:61 Flt Cutoff`, not as two `[CC]` entries.
- List everything.
  The only limits are the ones in `format.md` (8 pots, 64 automation lanes, 16 drum rows, 128 PCs); never drop entries to fit a limit you assume.
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
- Replace characters the Hapax rejects or cuts: `#` → `No` (`Tom No2`) or drop it, `&` → `+` or `and`, `°` → `deg`, accented letters → plain ones.

## Directives

- `TYPE DRUM` when the instrument plays separate voices on separate notes (drum machines, samplers in kit mode); `POLY` otherwise.
- `MPE`, `POLYAT` or `AFTR` only when the manual documents MPE or polyphonic aftertouch.
- `OUTPORT`, `OUTCHAN`, `INPORT`, `INCHAN` are `NULL` unless the user says how the instrument is connected: `NULL` keeps whatever the track already has.
  The instrument's factory channel is not a reason to set `OUTCHAN`.
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
For the same parameter, prefer the CC pair (`CC_PAIR:29:61`) or 14-bit NRPN when it has more resolution than the CC.

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
- Line 2: `Source:` and the manual's title and version, as given on its cover.
- Keep it short; setup notes belong in the report, not the file.
- Only name characters: a `;`, `&`, `%` or `[` in `[COMMENT]` rejects the whole file.
  Use commas and full stops.
