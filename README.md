# hapax-skills

AI skills for the Squarp Hapax.

## hapax-instrument

Point it at a manual or MIDI implementation chart and it writes a Hapax instrument definition: every CC, NRPN, program change and drum note the instrument receives, with short names, 8 pots and a set of automation lanes chosen for you.
It targets Hapax OS 3.21 unless you tell it otherwise.

### Install

**Claude Code plugin** — add this repository as a marketplace and install the plugin:

```
/plugin marketplace add claytron/hapax-skills
/plugin install hapax-skills@hapax-skills
```

**Other agents** — with Node.js installed, the [skills](https://github.com/vercel-labs/skills) CLI installs it into Claude Code, Cursor, Codex and other agents:

```sh
npx skills add claytron/hapax-skills
```

**From a clone** — clone the repository once; every install below points at the clone, so `git pull` updates the skill.

```sh
git clone https://github.com/claytron/hapax-skills.git ~/src/hapax-skills
```

**Claude Code** — link the skill into your personal skills folder, then restart Claude Code:

```sh
mkdir -p ~/.claude/skills
ln -s ~/src/hapax-skills/skills/hapax-instrument ~/.claude/skills/hapax-instrument
```

To install it for one project only, link it into that project's `.claude/skills/` instead.

**claude.ai and the Claude desktop app** — zip the skill folder and upload it under Settings → Capabilities → Skills:

```sh
cd ~/src/hapax-skills/skills && zip -r ../hapax-instrument.zip hapax-instrument
```

Uploads are copies: after `git pull`, zip and upload again to update.

**Other Agent Skills hosts** — install the `skills/hapax-instrument/` folder as a skill; it is a standard `SKILL.md` with a `references/` folder and needs no scripts or dependencies.

### Update

Claude Code plugin:

```
/plugin marketplace update hapax-skills
```

Other agents:

```sh
npx skills update hapax-instrument
```

From a clone:

```sh
git -C ~/src/hapax-skills pull
```

The skill moved to `skills/hapax-instrument/` on 2026-09-29; if your symlink points at the old `hapax-instrument/` path, re-create it.

### Use

> Make a Hapax instrument definition for my Novation Peak from this manual: peak-user-guide.pdf

### Validating

The skill checks its output by hand unless the `hapax` command from [hapax-tui](https://github.com/claytron/hapax-tui) is installed, in which case it validates against the real rules and fixes what it finds. hapax-tui also edits definitions in the terminal.
