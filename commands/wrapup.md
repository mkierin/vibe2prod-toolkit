---
name: wrapup
description: End-of-session close-out. One command saves all session state so nothing is lost. Updates the project TASKS.md, optionally appends a worklog entry, updates the project CLAUDE.md, captures durable lessons to memory, commits (never pushes), writes the handoff note, then prints a receipt of what was saved where. DRAFT v0.1, review before adopting.
type: skill
---

Usage: say "wrapup" or `/wrapup`.

Defaults: current project only; commit but NEVER push; write the optional worklog entry then show it; save and stop (no new tab). Overrides at the bottom.

## Principle
Important info, todos, decisions, and time worked must never get lost between sessions. Every piece of session state has exactly ONE permanent home. This skill harvests the session, routes each piece to its home, then prints a receipt so the save is verifiable. The receipt is the "nothing got lost" guarantee.

## Phase 1 — Detect scope
- Determine the project from cwd. Read its `CLAUDE.md`, `TASKS.md`, and (if present) `WORKLOG.md` at the project root. Compute today's date as YYYY-MM-DD (never write "today" or "yesterday").
- If cwd is not a recognized project, say so and ask which project; if none, write only the handoff note.

## Phase 2 — Harvest the session
Scan THIS conversation and extract STATE (not narrative) into buckets:
- Shipped: what actually changed (files, commits, deploys, artifacts).
- Task changes: items completed, new todos, items now blocked / waiting on someone.
- Decisions: choices made and the one-line why.
- Durable lessons: anything that should change future behavior (corrections, gotchas, incidents).
- Time: rough split of the work, useful if you keep a worklog.
- Next: the single most concrete next action with file paths or commands.

## Phase 3 — Route each piece to its home (in order, all additive)
1. **TASKS.md** (project): mark done items `- [x] YYYY-MM-DD: outcome`, add new todos in the user's format, update "Waiting on". UPDATE existing items, never duplicate. A past-dated event must never sit in Open.
2. **WORKLOG.md** (project, OPTIONAL): if the project already keeps a `WORKLOG.md`, append ONE dated entry under the current month describing the work done, in plain English. If the project has no such file, skip this step entirely — don't create one unasked.
3. **CLAUDE.md** (project): update "Current State" and/or "Decision Log" ONLY if state actually changed. This file stays the source of truth.
4. **Memory**: for each durable lesson, write or update a note in wherever you keep cross-session memory (e.g. a `feedback_*` / `project_*` file plus a one-line pointer in an index file), if you use such a system. Dedup against existing notes first. Skip if nothing durable, or if you don't keep this kind of memory.
5. **Git**: stage the work, SECRET-SCAN the staged set (abort the commit on any `.env` / `*.key` / `*.pem` / token / credential, and tell the user). Commit with a clear message summarizing WHAT changed. NEVER push. Follow the project's branch convention (some commit to main directly; branch first only if that is the project's rule). Co-author per the session's commit attribution rule, if any.
6. **Handoff**: write the note in the `/handoff` format (cwd on line 1) to BOTH `~/.claude/handoff/HANDOFF-<key>.md` (the Phase-1 project key, so `/pickup <key>` resumes the right thread) and `~/.claude/handoff/HANDOFF.md` (the latest pointer). Archive any existing version of each to `~/.claude/handoff/log/` first (`<key>-<timestamp>.md` and `<timestamp>.md`). Do NOT open a new tab (default).

## Phase 4 — Print the receipt
Compact table of what was saved and where, e.g.:
- TASKS.md: 3 done, 2 new, 1 waiting
- WORKLOG.md: entry for YYYY-MM-DD added (show the text) — or "skipped (no worklog file)"
- CLAUDE.md: Decision Log + Current State updated
- Memory: 1 new note (name) — or "skipped (no memory system)"
- Git: committed <hash>, N files, NOT pushed
- Handoff: written

Then a **Needs you** list: anything unresolved (push pending, a secret found and skipped, an open decision, a question waiting on someone). If a target had no changes, say "skipped (no change)".

## Safety
- Never push, never deploy.
- Secret scan before every commit.
- Additive edits only to TASKS / WORKLOG / CLAUDE. Never delete the user's content.
- Idempotent: safe to run twice (skip unchanged targets).

## Overrides (say these with the command)
- "wrapup all projects" -> Phase 1 covers every project touched this session.
- "wrapup no commit" / "wrapup stage only" -> Phase 3 step 5 stages only, no commit.
- "wrapup and open tab" / "wrapup continue" -> after the receipt, run `new-session` if it's installed (see `/new-session`); otherwise note it's not available.
- "wrapup draft worklog" -> show the entry and wait for OK before writing it.
- "wrapup and push" -> still requires an explicit confirm before the push.
