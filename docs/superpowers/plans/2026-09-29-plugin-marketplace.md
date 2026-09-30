# Plugin Marketplace Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the repo installable as a Claude Code plugin with `/plugin marketplace add claytron/hapax-skills`, while keeping `npx skills add` and the clone installs working.

**Architecture:** The repo root is both a one-plugin marketplace and the plugin itself (`"source": "./"`), the same layout as `home-assistant-skills` and `ponytail`.
Claude Code discovers skills under `skills/`, so `hapax-instrument/` moves to `skills/hapax-instrument/`.
`plugin.json` has no `version`, so Claude Code tracks the commit SHA and every push is an update.

**Tech Stack:** Claude Code plugin manifests (JSON), `claude plugin validate`, the `skills` CLI (`npx skills`).

**Spec:** None; design agreed in conversation on 2026-09-29.

## Global Constraints

- Plugin name and marketplace name are both `hapax-skills`, so the install id is `hapax-skills@hapax-skills` and the skill appears as `hapax-skills:hapax-instrument`.
- No `version` field in `plugin.json` or `marketplace.json`.
- Skill content (`SKILL.md`, `references/`) is moved unchanged.
- Historical docs (`docs/superpowers/**`, `eval/RESULTS.md`) keep their old `hapax-instrument/` paths; they record what was true then.
- Markdown: one sentence per line; files end with a newline, no trailing whitespace.

## Review Focus

- `npx skills add claytron/hapax-skills` must still find `hapax-instrument` after the move; checked with `npx skills add . --list` in Task 1.
- `npx skills update hapax-instrument` for people who installed before the move: the skill name is unchanged, so it should resolve; checked by the same `--list` run showing the name `hapax-instrument`.
- Clone users with a symlink to `~/src/hapax-skills/hapax-instrument` get a dangling link after `git pull`; the README Update section tells them to re-link (Task 2).
- `claude plugin validate --strict` must pass on the root, so unknown fields or a missing owner fail loudly (Task 1).
- The skill must actually load in a session from the plugin, not just validate (Task 1, `--plugin-dir` check).

---

### Task 1: Move the skill and add the manifests

**Files:**
- Move: `hapax-instrument/` → `skills/hapax-instrument/`
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`

**Interfaces:**
- Produces: install id `hapax-skills@hapax-skills`; skill path `skills/hapax-instrument/` (Task 2 documents both).

- [ ] **Step 1: Confirm validation fails before the change**

Run: `claude plugin validate --strict .`
Expected: FAIL (no manifest found).

- [ ] **Step 2: Move the skill**

```sh
mkdir -p skills
git mv hapax-instrument skills/hapax-instrument
```

- [ ] **Step 3: Write `.claude-plugin/plugin.json`**

```json
{
  "name": "hapax-skills",
  "description": "AI skills for the Squarp Hapax: generate instrument definitions from synth and drum machine manuals.",
  "author": {
    "name": "Clayton Parker",
    "url": "https://github.com/claytron"
  },
  "repository": "https://github.com/claytron/hapax-skills"
}
```

- [ ] **Step 4: Write `.claude-plugin/marketplace.json`**

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "hapax-skills",
  "description": "AI skills for the Squarp Hapax.",
  "owner": {
    "name": "Clayton Parker",
    "url": "https://github.com/claytron"
  },
  "plugins": [
    {
      "name": "hapax-skills",
      "description": "Generate Squarp Hapax instrument definitions from synth and drum machine manuals.",
      "source": "./"
    }
  ]
}
```

- [ ] **Step 5: Validate**

Run: `claude plugin validate --strict .`
Expected: PASS, with the marketplace, the plugin and one skill (`hapax-instrument`) reported.
If `--strict` flags a field above as unrecognized, remove that field rather than suppressing the check.

- [ ] **Step 6: Check the skill loads from the plugin**

Run: `claude -p --plugin-dir . "List the names of the skills available to you that contain 'hapax'. Names only."`
Expected: output includes `hapax-skills:hapax-instrument`.
(The user also has a personal `hapax-instrument` skill linked from `~/.agents/skills`; it may appear too, unprefixed. Only the prefixed one proves the plugin works.)

- [ ] **Step 7: Check the `skills` CLI still finds it**

Run: `npx -y skills add . --list`
Expected: lists `hapax-instrument`.

- [ ] **Step 8: Commit**

```sh
git add -A skills .claude-plugin
git commit -m "Package the repo as a Claude Code plugin marketplace"
```

### Task 2: Document the plugin install

**Files:**
- Modify: `README.md` (Install and Update sections)

**Interfaces:**
- Consumes: install id `hapax-skills@hapax-skills`, skill path `skills/hapax-instrument/` from Task 1.

- [ ] **Step 1: Add the plugin install as the first Claude Code option**

At the top of `### Install`, before **Quick install**, add:

````markdown
**Claude Code plugin** — add this repository as a marketplace and install the plugin:

```
/plugin marketplace add claytron/hapax-skills
/plugin install hapax-skills@hapax-skills
```
````

Rename the existing **Quick install** label to **Other agents** and change its lead-in to say the `skills` CLI installs into Claude Code, Cursor, Codex and other agents.

- [ ] **Step 2: Fix the clone paths**

In the clone instructions, change every `~/src/hapax-skills/hapax-instrument` to `~/src/hapax-skills/skills/hapax-instrument`.
Change the zip command to:

```sh
cd ~/src/hapax-skills/skills && zip -r ../hapax-instrument.zip hapax-instrument
```

Change the **Other Agent Skills hosts** line to name the `skills/hapax-instrument/` folder.

- [ ] **Step 3: Update the Update section**

Add a first entry for the plugin:

````markdown
Claude Code plugin:

```
/plugin marketplace update hapax-skills
```
````

Rename the existing "Quick install:" label to "Other agents:".
Under "From a clone:", after the `git pull` block, add:

```markdown
The skill moved to `skills/hapax-instrument/` on 2026-09-29; if your symlink points at the old `hapax-instrument/` path, re-create it.
```

- [ ] **Step 4: Check every path in the README exists**

Run: `rg -o 'skills/hapax-instrument[^ )`]*' README.md | sort -u` and confirm each path exists under the repo (after stripping the `~/src/hapax-skills/` prefix).
Run: `rg -n '~/src/hapax-skills/hapax-instrument' README.md`
Expected: no matches.

- [ ] **Step 5: Commit and push**

```sh
git add README.md
git commit -m "Document installing as a Claude Code plugin"
git push
```

- [ ] **Step 6: Verify from GitHub**

Run: `claude plugin marketplace add claytron/hapax-skills && claude plugin install hapax-skills@hapax-skills`
Expected: both succeed.
Then remove them again (`claude plugin uninstall hapax-skills@hapax-skills`, `claude plugin marketplace remove hapax-skills`) unless the user wants to keep the plugin; if they keep it, they should `npx skills remove hapax-instrument` so the skill is not loaded twice.
