---
name: handoff
description: Write a project-specific session handoff note so the next session can resume the right thread. Saves ~/.claude/handoff/HANDOFF-<key>.md (key derived from cwd) plus HANDOFF.md as the latest pointer, archiving the previous one.
---

# Session Handoff

You are writing a handoff note for a fresh Claude session that will start with no memory of this conversation. The next session reads this file and continues from where you left off.

## Step 0: Skip if a fresh handoff already exists

Before doing any work, check whether a handoff for this cwd was already written in the last few minutes (e.g. the user ran `/handoff` twice, or is just re-launching a tab). Derive the key (Step 2) and run:

```bash
f="$HOME/.claude/handoff/HANDOFF-<key>.md"; [ -f "$f" ] && echo "exists, age(min): $(( ($(date +%s) - $(stat -c %Y "$f")) / 60 ))"
```

If the file exists, is younger than ~10 minutes, and nothing material changed since (no new commits, no new edits this turn), **do not rewrite it** — skip straight to Step 4 and just launch the tab. Tell the user you reused the existing handoff. Only rewrite when real new state has accumulated.

## Step 1: Snapshot the work

Before writing, gather:
- Current working directory (`pwd`)
- Git status (uncommitted? on what branch?) — run `git status -sb` if in a repo
- What we actually accomplished this session (not what we discussed — what shipped)
- What's open or partially done
- What the user said is next
- Any non-obvious context the next session needs (decisions made, paths discovered, gotchas)

If there are uncommitted changes related to completed work, remind the user before writing the handoff. Do not auto-commit.

## Step 2: Determine the project key, then archive

Derive the **project key** from cwd: the folder name under your projects root (e.g. cwd `.../projects/billing-service/...` → key `billing-service`). If cwd isn't inside a recognized project folder, there is no key — use the generic handoff only. This key is what `/pickup <key>` matches on when several projects run in parallel.

Archive before overwriting (timestamp = `date +%Y-%m-%d-%H%M%S`); never overwrite without archiving:
- If a key exists and `~/.claude/handoff/HANDOFF-<key>.md` exists, move it to `~/.claude/handoff/log/<key>-<timestamp>.md`.
- If `~/.claude/handoff/HANDOFF.md` exists, move it to `~/.claude/handoff/log/<timestamp>.md`.

## Step 3: Write the handoff

Write the SAME note to both (so parallel projects don't clobber each other, and the launcher still finds the latest):
- `~/.claude/handoff/HANDOFF-<key>.md` — the project-specific handoff `/pickup <key>` reads. Skip if there's no key.
- `~/.claude/handoff/HANDOFF.md` — identical content, the "latest session" pointer the `new-session` launcher and a bare `/pickup` read.

Use this exact format (cwd on line 1):

```markdown
cwd: <absolute path the next session should cd into>
date: <YYYY-MM-DD HH:MM>
project: <short label, e.g. "api refactor" or "billing service">
branch: <git branch if relevant, else omit>

## What we did
- bullet 1
- bullet 2

## Current state
- where things stand right now (running processes, half-applied edits, open files, blockers)
- uncommitted changes? say so explicitly

## Next step
- the single most concrete next action, with file paths or commands

## Context the next session needs
- non-obvious decisions made this session
- paths/IDs/URLs that matter
- anything the next Claude would waste tokens rediscovering

## Don't
- things we already tried and ruled out
- approaches the user rejected
```

The `cwd:` line at the top is parsed by the `new-session` launcher — keep it on line 1, format exact.

## Step 4: Launch the new tab (default behavior, if the launcher is installed)

The `new-session` launcher (shipped in `bin/` of this toolkit) is OPTIONAL — it's WSL + Windows Terminal + zellij specific. After writing the handoff:

- **If `new-session` is on PATH**, run it to spawn a fresh tab → zellij → claude (or codex) with the handoff auto-loaded. This is the default when available — the whole point of `/handoff` is to hand off, so do it in one step.

```bash
new-session
```

- **If `new-session` is NOT on PATH**, skip the launch — just report that the handoff was written and its path, and tell the user to open their next session manually and read it (or run `/pickup`).

Skip the launch only if the user explicitly said one of:
- "just write it" / "don't open a tab" / "no new session"
- "handoff and exit" (they want to close current session, not open a new one)
- "handoff to clipboard" (copy mode, no launch)

Flag translation (only relevant if the launcher is installed):
- "handoff here" → `new-session --here`
- "handoff fresh" → `new-session --fresh`
- "handoff to codex" / "codex handoff" / "handoff codex" / mentions of "codex" → `new-session --codex`
- "handoff to claude" (explicit) → `new-session --claude` (the default)

Flags compose: e.g. "handoff to codex here" → `new-session --here --codex`.

## Step 5: Confirm

Tell the user (in 2-3 lines max):
- Handoff written (with path)
- New tab opening (or not, with reason — including "launcher not installed")
- One-line next-step summary so they remember without reading the file

## Rules

- **Be concrete, not narrative.** "Edited `foo.py:42` to fix the auth flow" beats "we worked on authentication".
- **No fluff sections.** If "Don't" is empty, omit it. Same for any section.
- **Under 60 lines total.** The next session reads this cold — long handoffs waste context.
- **Quote file paths exactly.** The next session uses these verbatim.
- **No summarizing the conversation.** Summarize the *state*, not the dialogue.
