# Skill test results

## Baseline

Run 2026-09-28, general-purpose subagents without the skill, prompts from `prompts.md`.

### Peak

```text
eval/baseline/Novation_Peak.txt  2 errors, 20 warnings
  W line 17: only the first 15 characters are shown: 'Osc1 ModEnv2>Pi'
  W line 42: only the first 15 characters are shown: 'Osc2 ModEnv2>Pi'
  W line 44: only the first 15 characters are shown: 'Osc2 ModEnv1>Sh'
  W line 47: only the first 15 characters are shown: 'Osc3 ModEnv2>Pi'
  W line 51: only the first 15 characters are shown: 'Osc3 ModEnv1>Sh'
  W line 55: only the first 15 characters are shown: 'Filt AmpEnv Dep'
  W line 56: only the first 15 characters are shown: 'Filt ModEnv1 De'
  W line 88: only the first 15 characters are shown: 'Osc1 ModEnv1>Sh'
  W line 106: only the first 15 characters are shown: 'Osc1 Saw Densit'
  W line 107: only the first 15 characters are shown: 'Osc1 Dens Detun'
  W line 113: only the first 15 characters are shown: 'Osc2 Saw Densit'
  W line 114: only the first 15 characters are shown: 'Osc2 Dens Detun'
  W line 120: only the first 15 characters are shown: 'Osc3 Saw Densit'
  W line 121: only the first 15 characters are shown: 'Osc3 Dens Detun'
  W line 135: only the first 15 characters are shown: 'ModEnv1 Velocit'
  W line 137: only the first 15 characters are shown: 'ModEnv2 Velocit'
  W line 143: only the first 15 characters are shown: 'LFO1 Fade In/Ou'
  W line 149: only the first 15 characters are shown: 'LFO2 Fade In/Ou'
  W line 167: only the first 15 characters are shown: 'Reverb High Pas'
  W line 181: only the first 15 characters are shown: 'ModMatrix Selec'
  E line 272: ';': not allowed in names
  E line 275: ';': not allowed in names

1 file, 2 errors, 20 warnings — Hapax OS 3.21
```

- Rejected character in `[COMMENT]`: `;` on lines 272 and 275 — the file does not load.
- 20 names over 15 characters (`Osc1 ModEnv2>Pitch`, `Reverb High Pass`, `ModMatrix Select`...).
- 14-bit CC pairs (Filter Frequency 29,61; osc coarse/fine; LFO rates) listed as their MSB in `[CC]`; no `[CC_PAIR]`; pot 1 on 7-bit `CC:29`.
- Invented a limit: "under 128" NRPNs, so mod matrix slots 9–16 were dropped.
- Guessed `OUTPORT A`, `OUTCHAN 1` from the factory channel.
- `[COMMENT]` claims bipolar CCs are "centred at 64", but no defaults are set.
- `[AUTOMATION]` is reasonable (40 lanes, continuous); spot check of 10 CCs against pp. 40–43 matched.

### TR-8S

```text
eval/baseline/TR-8S.txt  3 errors
  E line 8: unknown directive DEFAULT_NOTE
  E line 9: unknown directive DEFAULT_PATTERN
  E line 100: ';': not allowed in names

1 file, 3 errors, 0 warnings — Hapax OS 3.21
```

- Invented directives `DEFAULT_NOTE`, `DEFAULT_PATTERN` — the file does not load.
- Rejected character in `[COMMENT]`: `;`.
- Believed the Hapax has 8 drum lanes; dropped MT, CC and RC (16 from 3.10).
- `[AUTOMATION]` left empty.
- Empty sections written out (`[PC]`, `[NRPN]`, `[AUTOMATION]`).
- Correct: transmit-only CC 14 and 70 skipped; `TYPE DRUM`; `ROW:NULL:NULL:NOTE`; pots on continuous controls.

## With the skill

Run 2026-09-28, same prompts, general-purpose subagents told to follow `hapax-instrument/SKILL.md`.
Peak ran with `hapax` on PATH; TR-8S was told it was not installed (self-check path).

### Peak

```text
eval/skill/Novation_Peak.txt  OK

1 file, 0 errors, 0 warnings — Hapax OS 3.21
```

- Fixed: no rejected characters; no names over 15 characters; 14-bit pairs in `[CC_PAIR]` (16), pots 1, 5, 6 on `CC_PAIR:`; all 16 mod matrix slots kept; ports `NULL`; defaults only where the chart states `64 (0)`.
- Spot check of 20 entries (every tenth CC, CC pair and NRPN) against pp. 40–43: all match.
- `[AUTOMATION]`: exactly 64, continuous parameters; dropped groups (coarse/fine pitch, drift, key tracking, LFO sync/fade/slew, mod matrix depths) named in the report.
- Pots: cutoff, resonance, env amount, osc shape, LFO1 rate, LFO1>filter, reverb level, amp release — all continuous.
- `TYPE POLYAT`: the manual documents receiving polyphonic aftertouch (p. 34). On 3.10 this is 1 error plus 33 `default.ignored` warnings; expected for a 3.21 target, and SKILL.md step 5 asks the OS when POLYAT is involved.
- No duplicate names within 15 characters; no `[PC]` (manual lists no patch names).

### TR-8S

```text
eval/skill/TR-8S.txt  OK

1 file, 0 errors, 0 warnings — Hapax OS 3.21
```

- Fixed: no invented directives; no rejected characters; 11 drum lanes in panel order (rows 9–11, 3.10+); 53 automation lanes; empty sections left out.
- Recognized-only: CC 14 and 70 skipped.
- Self-check path: report says the file was checked by hand and suggests hapax-tui.
- Also clean on 3.10.

### Image-only chart

Pass: `Peak_Image.txt` built from `peak-chart.png` (one page), 0 errors, 0 warnings; the report says the page ends at "(Continues...)" and more pages are needed.
