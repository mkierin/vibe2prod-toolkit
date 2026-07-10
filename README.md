# vibe2prod toolkit

An **index + installer** for a growing set of standalone [Claude Code](https://claude.com/claude-code) tools — the custom commands and gates I use to take vibe-coded projects to production without the "it works on my machine" lie.

This repo doesn't bundle the tools. It holds a [`manifest.json`](manifest.json) that points at each tool's **own GitHub repo**, and an installer that pulls only the ones you pick. Each tool is independent — you can install it straight from its own repo, or grab several at once through here.

> Built and dogfooded on the **vibe2prod** channel → [watch on YouTube](https://www.youtube.com/channel/UCm8MSB6z3D4ViAuJc1qvzTw)

## Tools

| Tool | Repo | What it does |
|------|------|--------------|
| **reality-check** | [mkierin/reality-check](https://github.com/mkierin/reality-check) | Definition-of-Done gate — prove a change works with real data before you claim done or deploy. Ships a deploy-blocking hook + an advisory "you claimed done with no proof" nudge. |
| **spec2prod** | [mkierin/spec2prod](https://github.com/mkierin/spec2prod) | Turn a build session into a runnable SPEC.md a cold agent can one-shot — /spec-capture + /spec-distill + session-index skills. |
| **animated-diagram** | [mkierin/animated-diagram](https://github.com/mkierin/animated-diagram) | Turn a static infographic into an animated, interactive 16:9 page for your videos — one prompt; ships a presenter-stepped deck assembler. |
| **core session commands** | *(in this repo, being split out)* | `/handoff`, `/handoff-many`, `/wrapup`, `/pickup`, `/new-session`, `/debug`, `/postmortem`, `/fix-skill` — session-continuity + general workflow commands. |

More tools get added as single lines in `manifest.json`; the installer picks them up automatically.

## Install (guided)

Paste this **one line** into a Claude Code session:

> Read https://raw.githubusercontent.com/mkierin/vibe2prod-toolkit/master/install.md and act as the installer it describes. Print the banner first, then show the menu.

Claude prints the banner, reads the tool index, and shows a menu. It installs only what you pick — clones each selected tool from its own repo, copies its commands/hooks into `~/.claude/`, wires any hooks into `settings.json` (with a backup, never clobbering), and can optionally set up a sane permission profile. Nothing runs without your confirmation.

*(Prefer to do it by hand? Paste [`install.md`](install.md) directly — same result.)*

## Install a single tool by hand

Each tool repo is self-contained and carries its own `install.json` + README. To install just one, clone it and follow its README, e.g.:

```bash
git clone https://github.com/mkierin/reality-check.git
# then copy commands/ and hooks/ into ~/.claude/ per its README
```

## How the index works (for contributors / future me)

- **`manifest.json`** — the index. A list of tools, each with `id`, `repo`, `branch`, `install` (path to that repo's install.json), and `desc`. Adding a tool = one entry here.
- **Each tool repo** ships an **`install.json`** declaring what to install:
  - `commands` — `.md` files copied into `~/.claude/commands/`
  - `hooks` — `files` copied into `~/.claude/hooks/`, plus `settings` entries merged into `~/.claude/settings.json`
  - `bin` — optional launcher scripts (asked about, never forced)
  - `skills` — folders copied into `~/.claude/skills/` (each entry has `src` + `dest`; Claude Code discovers `SKILL.md` automatically)
- **`install.md`** — the installer instructions the model follows. It reads the manifest, never a hard-coded list, so it never goes stale.

This is deliberately a migration in progress: the "core session commands" still live in this repo for now and get installed as one bundle. As each is split into its own repo, it moves from that bundle to its own `manifest.json` line — no installer changes needed.

## License

MIT — see each tool's repo for its own license.
