---
name: postmortem
description: "Run an incident postmortem that CLOSES THE LOOP — root-cause an incident, then convert each root cause into a durable, automated control in the project itself (a test, a lint rule, a check script, a checklist entry). A postmortem isn't done until at least one root cause produced a real automated check. Say 'postmortem', 'do a postmortem', 'write up that incident', or run after any prod/runtime incident."
---

This skill exists to make incidents pay for themselves: every root cause should leave behind something that automatically prevents its recurrence, not just a written account of what happened. If a postmortem ends in only a doc and "be more careful," it failed.

Two modes. `init` runs the analysis and writes the doc. `route` converts findings into controls. Run them in sequence; `route` is mandatory before the postmortem is "done".

---

## Mode: init — root-cause and document

### Step 1: Establish the facts
Gather from the conversation/incident (ask only for what's missing):
- **What broke**, observable symptom first, then the actual fault.
- **Detection**: how was it found? (If a human noticed a downstream symptom rather than an alert firing, that itself is a finding — see Step 4 "detection gaps".)
- **Window**: start → resolution, in UTC, absolute dates (never "today"/"yesterday").
- **Impact**: real-money / data-loss / outage / user-facing? Quantify.

### Step 2: Root causes (one section each)
For each independent fault, write: the mechanism (how it actually failed, quote `file:line`), and **why it was possible** (the missing control, not just the bug). Separate compounding causes — most incidents have 2–3. Distinguish:
- **Code fault** (a bug a reviewer/lint should have caught)
- **Process gap** (a feature shipped without closing its operational loop — no retention, no reader, no test, no alert)
- **Silent-failure gap** (the failure produced no signal: crash logged to a file nobody reads, a job that does nothing, exit 0 on a lie)
- **Runtime-guardrail gap** (no disk/health/heartbeat monitor)

### Step 3: Write the doc
Use the project's existing convention. Detect it: look for `docs/postmortem-*.md`, `postmortem-*`, or an `incidents/` folder; match that filename/format. If none exists, create `docs/postmortem-YYYY-MM-DD-<slug>.md`. Sections: Summary, Impact, Timeline (UTC), Root Causes, Detection, Resolution, **How this could happen + how to prevent it when implementing features**, Action Items table.

The prevention section is the load-bearing one. Organize it by the moment a future feature would introduce the same class: "when you add a table / a scheduled job / parameterized query / …". Generic advice ("test more") is banned; every item must name the concrete trigger and the concrete control.

---

## Mode: route — convert each root cause into a durable control

For EVERY root cause, pick exactly one home and **write the change** (not a recommendation). Routing table:

| Root-cause class | Durable home | Form of the control |
|---|---|---|
| Deterministic bad-code pattern (grep/AST-detectable) | the project's lint config or a `scripts/check_*.py` | add a rule to the linter, or a runnable check that exits non-zero on the pattern |
| Process gap in how features ship | the project's own contribution/checklist doc (e.g. `CONTRIBUTING.md`, a PR template, or a `docs/FAILURE-MODES.md`) | append a checklist item or a new gate in the review process |
| Project-specific trap | that project's failure-modes catalog | append an entry (WRONG vs RIGHT); create `docs/FAILURE-MODES.md` if absent |
| My (assistant) behavioral mistake | wherever you keep cross-session notes/memory, if you use such a system | a note with **Why** + **How to apply** |
| Missing runtime guardrail | an ops action item with an owner + a tracking line | a concrete monitor/alert task, not "add monitoring" |

Rules for routing:
- **Prefer the most automated home available.** A runnable check or test beats a checklist entry beats a note beats an action item. Push each cause as far up that ladder as it can go.
- **A new failure mode that no existing check catches is itself a finding** — say so explicitly and propose where the check should live.
- When you add a `check_*.py` or a test, **prove it works**: run it (should pass on current code) AND demonstrate it catches the bug (run it against the pre-fix version, a `.bak`, or a crafted fixture). Paste both results.
- Mirror any prod hotfix into the source repo and commit; never leave a fix that the next deploy clobbers.

### The theater-gate (mandatory)
After routing, state plainly:

> **Theater-gate: PASS** — root cause [X] produced automated check `<path>` (proven to catch the pattern).
> or
> **Theater-gate: FAIL** — no root cause produced an automated check; this postmortem only generated docs/notes. The most automatable cause is [Y]; build its check before calling this done.

If FAIL, do not report the postmortem as complete. Build the check.

### Log it
Append one row to the postmortem log (create if absent) at the project root or `docs/POSTMORTEM-LOG.md`:

```
| Date | Incident | Root causes | Controls emitted (check/test/checklist/note) | Theater-gate |
```

This log is the evidence the loop works.

---

## Rules
- UTC, absolute dates everywhere. No "today"/"yesterday".
- Quote `file:line` for every mechanism claim; don't assert a cause you haven't read.
- Real-money / live-trading / production-critical incidents: confirm the fix against live behavior before claiming resolved (pair with a verification/reality-check pass if you have one).
- Project-agnostic: works for any codebase. Detect the project's conventions; don't impose new ones.
- Don't double-route: each cause lands in exactly one home (cross-link from the others if relevant).

## Trigger phrases
- "do a postmortem" / "write up that incident" / "postmortem this"
- "how did this happen and how do we prevent it"
- after resolving any prod/runtime incident

## Companion skills
- `/fix-skill` — same loop, but for skill errors (skill gap → patch the skill).
- If you have a "verify before deploy" skill, run it to prove the fix works before calling the incident resolved.
