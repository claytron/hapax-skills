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
