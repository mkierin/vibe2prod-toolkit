---
name: pickup
description: Morning pickup — restore where you left off. With a project key (e.g. /pickup myapp) it reads that project's HANDOFF-<key>.md plus its CLAUDE.md/TASKS.md/PLAN.md (never another project's handoff); bare /pickup uses the latest handoff. Optionally layers in a task planner/kanban if you have one. Mirror of /wrapup and /handoff. Read-only.
---

# Morning Pickup

The user is starting their day. Reorient them fast: fuse "what was I mid-doing" (the
last session's handoff) with "what's actually on today" (the project's own task file,
and optionally a planner if the user has one), then point at one concrete first action.
This is the resume side of `/handoff` and `/wrapup`.

Keep it tight. This is a briefing, not a wall of text. Read-only — never edit,
commit, or write anything. Use the real system date everywhere (never "today" as a
literal — compute it).

Optional `$ARGUMENTS`: a project key (focus the briefing on that project) or a
handoff name (e.g. `bot-data-exports`) to pick up a specific parked thread.

## Step 1 — restore context (project-scoped when an arg is given)

**If `$ARGUMENTS` names a project key** (e.g. `myapp`) — the common case with several
projects running:
- Read the project's persistent spine FIRST — `projects/<key>/CLAUDE.md` + `TASKS.md` +
  `PLAN.md` (adjust the path to wherever your projects root keeps that project). This is
  the source of truth and is always current (it's what `/wrapup` updates).
- Then read `~/.claude/handoff/HANDOFF-<key>.md` if it exists, for the "where I stopped
  mid-task" layer.
- **Never** fall back to the generic `HANDOFF.md` or another project's handoff for a
  project pickup. If there's no `HANDOFF-<key>.md`, just brief from the folder spine and
  say "no mid-task handoff — briefing from project docs."

**If no arg** (bare `/pickup`): prefer `HANDOFF.md`; if absent pick the most recent
`HANDOFF*.md` (`ls -t ~/.claude/handoff/HANDOFF*.md`). If several exist, read the newest
and LIST the others by name + date so the user can pick a thread. If none exist, say so
and skip to Step 2 (task-file-only briefing).

From the handoff (when present) pull: `cwd`, `project`, the **Next step**, **Current
state** (esp. "uncommitted changes?"), and the **Don't** list. If it says work was
uncommitted, `cd` to its `cwd` and run `git status -sb` — surface anything still
uncommitted as a flag (do NOT commit it).

## Step 2 — what's on today

Read the project's own `TASKS.md` directly and compute **overdue** and **due today**
from it — this works standalone, with no other tooling required.

**Optional planner aside:** if you have a task server/planner running (e.g. a local
kanban dashboard), you can additionally pull today's time-blocks and calendar from it
and merge that in. This is a nice-to-have, not a dependency — skip it entirely if you
don't have one, and don't block the briefing waiting on it.

## Step 3 — the briefing (output)
Print, in this order, compact:

```
## Good morning — <weekday, YYYY-MM-DD>

Where you left off
- <project>: <next step from handoff, one line>
- <flag: N uncommitted files in <cwd>, if any>

On your calendar today        (omit if you don't have calendar/planner integration)
- <HH:MM> <meeting> … <free hours left>

Scheduled for today
- <project> · <task> <effort>      (from TASKS.md, or your planner if you have one)
- … or: "nothing explicitly scheduled — here's what's overdue/open instead"

Overdue / due today
- ⚠ <project> · <task> <date>
- …

Start here
- <the single best first action>
```

**Choosing "Start here":** reconcile the handoff's Next step against what's overdue.
If something is overdue and client- or deadline-facing, it usually outranks a parked
personal thread — say so in one line. Recommend ONE path, don't list options.

## Step 4 — offer to resume (then stop)
End with a single offer, e.g.:
"Want me to `cd <cwd>` and pick up <next step>, or open a fresh session for it?"

If the user says yes: `cd` into the chosen project, read its `CLAUDE.md` + `TASKS.md`
first (project state lives there), then continue. Don't auto-start work before the
user picks — this step is the only place pickup hands control back.

## Notes
- If you have a project-status or portfolio-style skill of your own, mention it here
  as an option for anyone wanting a wider budget/capacity view — this skill itself
  doesn't need one.
- Don't re-run a full status dump unless asked; this is a focused start, not a full audit.
