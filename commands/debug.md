---
name: debug
description: "Structured debugging flow — reproduce → isolate layer → 3 hypotheses → instrument and verify → fix → regression test. Works for any domain (Python, JS, etc). Never skips steps."
---

# /debug

Structured path from "something is broken" to "fixed and won't regress."
`$ARGUMENTS` = what is broken (error message, symptom, feature name).
If empty, infer from the most recent error or change in context.

**Gate rule:** each step requires real evidence before proceeding. No skipping.
No "probably" fixes. No declaring done without a passing regression test.

---

## Step 1 — Claim

State precisely:
- **Expected:** what SHOULD happen (one sentence)
- **Actual:** what DOES happen — paste the exact error, wrong value, or symptom
- **Scope:** first occurrence (when did it start?), frequency (always / sometimes / once?)

If you can't write "Expected" and "Actual" concretely, stop and ask the user to clarify.
Vague claims produce vague fixes.

---

## Step 2 — Reproduce

Actually trigger the bug. Don't theorize — run it.

- Run the failing path: execute the script, call the endpoint, reload the app, re-run the query
- Paste the exact output (error message + stack trace, or wrong value + context)
- Confirm the bug is reproducible. If it only happens intermittently, note the reproduction rate and conditions

**Gate:** if you cannot reproduce it, say so explicitly. Do NOT proceed to Step 3 with "I assume the bug is..." A bug you can't reproduce cannot be safely fixed.

---

## Step 3 — Isolate the layer

Identify WHICH layer the failure is in. Don't assume — check the boundary.

Use the layer map for the detected domain:

| Domain | Layers (check from data inward → output outward) |
|---|---|
| **Python** | Data source / input → Transform / business logic → API / interface → Config / env |
| **JavaScript / Frontend** | Data fetch → State management → Render → Style / layout → Browser API |
| **Generic** | Input / data → Processing / logic → Output / render → Config / environment |

For each layer boundary, ask: "is the data correct going IN to this layer?"
- Check one boundary at a time, starting from the data source
- The failing layer is the first one where data goes in correct and comes out wrong

Paste the evidence that identifies the layer (a log line, a query result, an intermediate value, a diff).

---

## Step 4 — Three hypotheses

Form exactly 3 hypotheses, ordered by likelihood (most likely first). Base them on:
- What the layer isolation found
- Recent changes (git diff, recent deploys, data updates)
- Known failure modes for this domain

Format:
```
H1 (most likely): <specific claim — e.g. "the loader is silently dropping NULLs because of a filter added in commit X">
H2: <alternative — different root cause, same symptom>
H3 (least likely but worth ruling out): <edge case or environmental cause>
```

Do NOT jump to the fix yet. Writing 3 hypotheses forces you to consider alternatives and
prevents locking onto the most familiar explanation.

---

## Step 5 — Instrument and verify

Test H1 first. Add the minimum instrumentation to confirm or refute:
- A `log()` / `print()` / `TRACE` before and after the suspected line
- A targeted query/filter to isolate suspect rows or records
- A `git stash` to test without a recent change
- A direct query to the data source to verify what's actually there

Run it. Paste the result.

**If H1 confirmed:** move to Step 6.
**If H1 refuted:** test H2 with the same rigour. Repeat for H3.
**If all 3 refuted:** go back to Step 3 — your layer isolation was wrong. Form 3 new hypotheses from the corrected layer.

Never mark a hypothesis as "confirmed" without pasted evidence.

---

## Step 6 — Fix

Now write the fix — no earlier.

- Change the minimum required to fix the root cause (not the symptom)
- Don't refactor surrounding code while fixing — that's a separate commit
- If the fix touches more than one file, say why each file is necessary

Verify the fix by re-running the reproduction case from Step 2.
Paste the output showing the bug is gone.

---

## Step 7 — Regression test

Write a test that would have caught this bug. This is mandatory, not optional.

- Unit test: if the bug is in a pure function or calculation
- Integration test: if the bug is at a layer boundary (data in → data out)
- Smoke check: if the bug is in a UI flow or end-to-end path

The test must:
1. Fail on the broken code (prove it)
2. Pass on the fixed code (prove it)

If the domain has no test framework, write the manual verification steps as a runnable checklist instead, and add it to the project's TASKS.md as a recurring check.

Paste the test and its output.

---

## Step 8 — Verdict

```
ROOT CAUSE: <one sentence>
FIX: <what changed>
REGRESSION TEST: <test name / file, or checklist location>
OPEN RISK: <anything NOT verified — edge cases, related paths, env-specific behavior>
```

Then commit: fix + test in the same commit. Message format:
`fix(<scope>): <what was wrong> — <root cause one phrase>`

---

## Hard rules

- **Never skip Step 2** — a fix for an unconfirmed bug is a guess
- **Never skip Step 7** — a fix with no test will regress
- **Never call it done** without pasting the post-fix reproduction output
- If you're halfway through and realize the scope is bigger than one bug (a design flaw, data corruption, systemic pattern), stop and surface it — don't silently expand the fix
- If you have a separate "verify before deploy" skill, run it as the deploy gate after this fix
