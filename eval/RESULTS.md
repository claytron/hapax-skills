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
