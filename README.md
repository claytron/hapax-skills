# hapax-skills

AI skills for the Squarp Hapax.

## hapax-instrument

Point it at a manual or MIDI implementation chart and it writes a Hapax instrument definition: every CC, NRPN, program change and drum note the instrument receives, with short names, 8 pots and a set of automation lanes chosen for you.
It targets Hapax OS 3.21 unless you tell it otherwise.

### Use

> Make a Hapax instrument definition for my Novation Peak from this manual: peak-user-guide.pdf

### Validating

The skill checks its output by hand unless the `hapax` command from [hapax-tui](https://github.com/claytron/hapax-tui) is installed, in which case it validates against the real rules and fixes what it finds.
The hapax-tui also allows for editing definitions in the terminal.

### Install

#### Claude Code plugin

Add this repository as a marketplace and install the plugin:

```text
/plugin marketplace add claytron/hapax-skills
/plugin install hapax-skills@hapax-skills
```

To update:

```text
/plugin marketplace update hapax-skills
/plugin update hapax-skills@hapax-skills
```

Then restart Claude Code.

#### claude.ai and the Claude desktop app

Zip the skill folder and upload it under Settings → Capabilities → Skills:

```sh
git clone https://github.com/claytron/hapax-skills.git ~/src/hapax-skills
cd ~/src/hapax-skills/skills && zip -r ../hapax-instrument.zip hapax-instrument
```

Uploads are copies: after `git pull`, zip and upload again to update.

#### Other agents

With Node.js installed, the [skills](https://github.com/vercel-labs/skills) CLI installs it into Claude Code, Cursor, Codex and other agents:

```sh
npx skills add claytron/hapax-skills
```

To update:

```sh
npx skills update hapax-instrument
```

#### From a clone

Clone the repository, then install the `skills/hapax-instrument/` folder as a skill in your agent, for example by symlinking it into the agent's skills folder so `git pull` updates it.
It is a standard `SKILL.md` with a `references/` folder and needs no scripts or dependencies.

```sh
git clone https://github.com/claytron/hapax-skills.git ~/src/hapax-skills
```

To update:

```sh
git -C ~/src/hapax-skills pull
```
