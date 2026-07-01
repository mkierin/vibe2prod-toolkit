---
name: new-session
description: Open a fresh Claude session in a new Windows Terminal tab (WSL → zellij → claude). No handoff note written, no summary of current session. Use when you just want a clean parallel session.
---

# New Session

Open a fresh Claude session in a new tab. Does NOT write a handoff note. Does NOT summarize the current session.

**Prerequisite:** this skill drives the `new-session` launcher script (shipped in `bin/` of this toolkit), which is WSL + Windows Terminal + zellij specific. If the launcher is not on PATH, tell the user it's not installed and stop — there's nothing else this skill can do standalone.

## What this does

Spawns a new Windows Terminal tab → WSL → zellij → claude with no handoff auto-load. **Default cwd is the current directory** (where you are now), so the new tab lands in the same project you're working in. Pass `from-handoff` to instead start in the cwd recorded in the latest handoff file.

## How to run

If the `new-session` launcher is on PATH, run it via Bash. Pick the flag based on what the user asked for:

- default → `new-session --here --fresh`  (fresh claude session **in the current dir** — matches where you are)
- "from last handoff" / "where I left off" → `new-session --fresh`  (cwd from last handoff or $HOME)
- "N tabs" / "--3" / "open 3" → `new-session --3`  (spawn N fresh tabs in the current dir; no handoff read)
- "worker" / "sonnet worker" → add `--worker`  (Sonnet at max effort — for implementation grunt work)
- user names a model/effort → add `--model M` and/or `--effort E` (low|medium|high|max)

```bash
new-session --here --fresh
```

That's it. Do not write to `~/.claude/handoff/HANDOFF.md`. Do not archive anything. Do not summarize this session.

If `new-session` is NOT on PATH, report that the launcher isn't installed (see the toolkit README's prerequisites) and stop.

## Confirm

One line: "→ new tab opened, fresh session." Optionally include the cwd it landed in (the launcher prints it).

## Rules

- **Never write a handoff** from this skill. If the user wants a handoff, they'll run `/handoff`.
- **Never auto-commit** or touch git state.
- **Do not run anything else** before/after the launcher. This is one command, one tab, done.
