---
name: handoff-many
description: Split the current session into multiple parallel handoff notes, one per task, then spawn N new tabs in parallel — one window per task. Use when you have several independent next steps and want to work on them in parallel sessions.
---

# Multi-Session Handoff

You are splitting the current session into multiple parallel handoff notes — one per task — so the user can pick up each task in its own window.

This is a TWO-PHASE interaction. Do NOT skip phase 1.

Note: spawning tabs automatically requires the optional `new-session` launcher (WSL + Windows Terminal + zellij, shipped in `bin/`). If it isn't installed, still do Phase 1 and Phase 2's file-writing steps, but skip the spawn step and instead tell the user the handoff files are ready to open manually.

## Parallel-team protocol (MANDATORY — these tabs run unattended, like an agent team)

Spawned tabs run with no human watching, so they must be NON-INTERFERING and AUTONOMOUS:

1. **One tab per working tree. One owner per file.** No two tabs may edit the same repo checkout
   or the same file. If a track works in a repo another track also touches, give it its OWN
   git worktree (pre-create it in Step 2b and set it as the handoff `cwd:`). A repo with a single
   owner-tab may use the main checkout. Partition files explicitly in each handoff ("you own X; do
   NOT touch Y — that's track Z").
2. **Autonomous: never ask the user.** The handoff must tell the agent to make sensible default
   decisions and keep going — do not pause for confirmation. Bake any needed decisions INTO the
   handoff so there's nothing to ask.
3. **Commit when done — never push/deploy.** Each tab commits its own work only. The orchestrator
   (this session) merges + deploys after all tabs report done. State this in every handoff.
4. **Report done via the status board + helper (NOT the Write/Edit tool).** Create one shared board
   `~/.claude/handoff/<BATCH>-STATUS.md` with one `PENDING` line per track. Each tab, when committed,
   marks itself done by running the Bash helper (covered by the `Bash(*)` allow, so it never prompts
   — even from inside a project repo, where the Write/Edit tool would hit the "outside project" guard):
   ```bash
   ~/.claude/handoff/team-status.sh ~/.claude/handoff/<BATCH>-STATUS.md <track> DONE <sha> <short note>
   ```
   The orchestrator polls the board (e.g. a backgrounded `until grep DONE` loop) and merges+deploys
   when every line is DONE. Each tab writes ONLY its own line.

## Phase 1: Survey and propose

Read the current session context and identify parallel work streams. Look for:
- Open tasks the user has explicitly mentioned
- Half-finished threads that could continue independently
- Logical splits in current work (e.g., "data pipeline" vs "frontend page template" vs "deploy config")
- Things gated on different inputs/decisions so they can run in parallel without conflict

Then present **3-7 candidates** as a numbered list. Each entry should be:
- Short title (3-6 words)
- One-sentence "what this thread is about"
- One-sentence "concrete next step"

Format (exact):

```
Here are candidate parallel tracks:

1. <title> — <thread description>
   next: <concrete next action>

2. <title> — <thread description>
   next: <concrete next action>

...

Which to split into separate sessions? (e.g. "1 3" or "1 2 4" or "all")
```

Then **STOP and wait** for the user's selection. Do not write any files yet.

## Phase 2: Generate handoffs and spawn tabs

Once the user replies with their selection (e.g. "1 2 4" or "1 3" or "all"):

### Step 2a: Create the status board + generate slug + handoff file

- Create the shared board `~/.claude/handoff/<BATCH>-STATUS.md` (pick a short BATCH name) with a
  header + one `- <slug>: PENDING` line per selected track.
- For each track: compute a short kebab-case slug; archive any existing `~/.claude/handoff/HANDOFF-<slug>.md`
  to `~/.claude/handoff/log/<timestamp>-<slug>.md`; write `~/.claude/handoff/HANDOFF-<slug>.md`:

```markdown
cwd: <absolute path — its OWN worktree if the repo is shared (Step 2b)>
date: <YYYY-MM-DD HH:MM>
project: <project label>
task: <slug>

## Isolation
- You are the SOLE writer of <this worktree/checkout>. Do NOT touch <other tracks' repos/files>.
- You own: <files/areas>. Off-limits: <files owned by other tracks>.

## What we did / Current state / Next step / Context
- task-scoped only (paths, decisions, gotchas). BAKE IN every decision — do not ask the user.

## When done (MANDATORY, autonomous — do not ask)
- Gate (build/test/lint as relevant), then COMMIT your work only. Do NOT push or deploy.
- Mark done via the helper (NOT the Write/Edit tool — it would prompt from inside a repo):
  `~/.claude/handoff/team-status.sh ~/.claude/handoff/<BATCH>-STATUS.md <slug> DONE <sha> <note>`
- Then stop. The orchestrator merges + deploys once all tracks are DONE.

## Don't
- Don't push/deploy. Don't ask the user. Don't edit another track's files/repo/worktree.
```

Each file is **scoped to its task** — strip context that doesn't apply. The whole point is each window gets a focused handoff, not a copy of the global state.

The `cwd:` line should match the directory most relevant to that task (e.g. `etl/` work cwd, `site/` work cwd).

### Step 2b: Pre-create isolated worktrees (for shared repos), then spawn one tab per task

**Before spawning**, guarantee no two tabs share a working tree: for each track whose repo is ALSO
used by another track, pre-create a dedicated git worktree and point that handoff's `cwd:` at it:
```bash
git -C <repo> worktree add -b <slug> /tmp/<slug> <base-commit-or-branch>
```
(Base off the right commit — e.g. a not-yet-deployed dependency commit, not stale main.) A repo used
by only ONE track can use its main checkout. Frontend worktrees lack `node_modules`, so prefer making
the single frontend track the sole owner of the main site checkout rather than a worktree.

If the `new-session` launcher is installed, call it with ALL the slugs as positional args in one
invocation. **Default for implementation worker tabs: `--worker` (= Sonnet at max effort)** — workers
do grunt implementation, the orchestrator stays on the expensive main model. Omit `--worker` only if
the user asked for a specific model, or pass `--model M --effort E` explicitly:

```bash
new-session slug-a slug-b slug-c --worker
```

This is one invocation, not N. The launcher loops internally and spawns one tab per slug.

If `new-session` is NOT installed, skip spawning — tell the user the handoff files are ready and
list the paths so they can open each in a manual session.

**Agent choice (codex vs claude)**: by default each tab runs `claude`. If the user said "to codex" / "codex" / "with codex" anywhere in the trigger or selection reply, append `--codex` to the new-session call so ALL spawned tabs use codex instead:

```bash
new-session slug-a slug-b slug-c --codex
```

Mixed-agent batches (e.g. "1 with codex, 2 with claude") are NOT supported in one call — ask the user to run the skill twice if they need that.

### Step 2c: Confirm

Report (max 5 lines):
- N handoffs written
- One line per slug: `<slug> → HANDOFF-<slug>.md → tab opened` (or "ready to open" if no launcher)
- Reminder: `each tab's claude will read its own handoff on boot`

## Rules

- **Never write handoffs in phase 1** — the user might not pick everything you suggested
- **Each handoff is task-scoped, not session-scoped** — the whole point is parallel focus
- **Slugs are kebab-case, ≤25 chars** — easier to type later via `new-session <slug>` for resuming
- **If user says "all"** select every numbered candidate
- **If user says invalid pick** (e.g. "1 99" when only 4 exist), ask again; do not silently drop
- **Under 50 lines per handoff** — same brevity rule as /handoff
- **Don't auto-commit** — same rule as /handoff

## When NOT to use this skill

- The user only has ONE concrete next task → use plain `/handoff`
- The user wants to fork without context loss → that's still `/handoff` (single)
- The candidate tracks are not actually independent (they share state) → recommend single `/handoff` and explain why splitting would create conflicts
