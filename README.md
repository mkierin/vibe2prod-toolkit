# vibe2prod toolkit

Session-continuity skills for [Claude Code](https://claude.com/claude-code). These are the
custom slash commands I use to keep long-running, multi-project work from losing state
between sessions: write a handoff before you stop, pick it back up cleanly next time,
close out a session properly, and a couple of general-purpose debugging/postmortem/
skill-maintenance commands.

Nothing here is project-specific — copy the commands in, adjust paths in your own
`TASKS.md`/`CLAUDE.md` files, and they work for any codebase.

## Skills

- **`/handoff`** — write a handoff note (what shipped, current state, next step, gotchas) so a fresh session can resume the right thread. Saves to `~/.claude/handoff/`.
- **`/handoff-many`** — split a session into several independent tracks, write one handoff per track, and (optionally) spawn one tab per track.
- **`/pickup`** — morning-briefing style resume: reads the last handoff plus the project's `TASKS.md`, tells you what's overdue, and recommends one first action. Read-only.
- **`/wrapup`** — end-of-session close-out: updates `TASKS.md`, optionally logs a worklog entry, updates `CLAUDE.md`, commits (never pushes), writes the handoff, prints a receipt.
- **`/new-session`** — open a fresh session in a new terminal tab. No handoff, no summary — just a clean parallel session.
- **`/debug`** — structured debugging flow: reproduce → isolate the layer → three hypotheses → instrument/verify → fix → regression test. Domain-agnostic.
- **`/fix-skill`** — after a correction or repeated mistake, find the skill file that should have prevented it and patch it (with your approval).
- **`/postmortem`** — root-cause an incident, then convert every root cause into an actual automated check (test, lint rule, checklist item) in the project — not just a written account.

## Quick install (guided)

Paste this **one line** into a Claude Code session:

> Read https://raw.githubusercontent.com/mkierin/vibe2prod-toolkit/master/install.md and act as the installer it describes. Print the banner first, then show the menu.

Claude fetches the installer, prints the Vibe2Prod banner, and shows a menu — it
installs only what you pick (the skills and/or a sane permission profile), and
nothing runs without your confirmation. *(Prefer to do it by hand? Paste
[`install.md`](install.md) directly instead — same result.)*

### Permission profiles

The installer can also set up a permission profile in `~/.claude/settings.json`:

- **Balanced (recommended)** — the three-list model: `allow` safe repetitive
  commands (builds, tests, read-only git), `ask` before destructive/outbound
  ones (`git push`, `rm`, publish), `deny` secrets outright (`.env`, `~/.ssh`).
  Fewest prompts without going blind. `deny` always wins over `allow`/`ask`.
- **Trusting** — `Bash(*)` + core tools. Fastest, but removes bash from safety
  checks entirely — sandbox/VM only.
- **Curated** — a hand-picked allowlist, no `ask`/`deny` layer.

It never writes secrets, warns if your existing file has a `Bash(*)` rule or a
leaked API key, and merges rather than clobbering your current settings.

## Manual install

```bash
git clone https://github.com/mkierin/vibe2prod-toolkit.git
cp vibe2prod-toolkit/commands/*.md ~/.claude/commands/
```

Optionally put the launcher scripts on your PATH:

```bash
cp vibe2prod-toolkit/bin/new-session ~/.local/bin/
cp vibe2prod-toolkit/bin/team-status.sh ~/.claude/handoff/team-status.sh   # handoff-many expects it here
chmod +x ~/.local/bin/new-session ~/.claude/handoff/team-status.sh
```

## Prerequisites

Everything in `commands/` works standalone — worst case, `/handoff` just writes a file
and tells you to open your next session manually.

The **`new-session` launcher** (used by `/handoff`, `/handoff-many`, and `/new-session`
to auto-spawn a new terminal tab with the handoff loaded) is optional, and specific to:

- WSL (Windows Subsystem for Linux)
- Windows Terminal (`wt.exe`)
- [zellij](https://zellij.dev/) as the terminal multiplexer inside WSL

If you're not on that stack, don't install `bin/new-session` — the skills detect it's
missing and fall back to just telling you the handoff path. You lose the auto-tab-spawn,
nothing else.

`bin/team-status.sh` is only needed if you use `/handoff-many` with multiple parallel
worker tabs reporting status back to a shared board.
