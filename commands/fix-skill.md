---
name: fix-skill
description: "Fix a skill after an error — analyze what went wrong, find the skill gap, patch it so it never recurs. Say 'fix-skill' after any rework, correction, or error pattern."
---

## Step 1: Capture the Error Pattern

Ask the user (or infer from conversation context): "What went wrong?"
- What was the incorrect output?
- What was the correct output?
- What type of work was it? (e.g. a script, an API integration, a piece of content, a config change)

## Step 2: Identify the Root Cause

Analyze WHY the error happened:
- Was it a syntax/function error? (wrong function name, wrong syntax)
- Was it a logic error? (correct syntax but wrong approach)
- Was it a process error? (skipped a step, forgot a constraint)
- Was it a context error? (missing project-specific knowledge)

## Step 3: Find the Responsible Skill

Search for which skill SHOULD have prevented this error:
1. Read `~/.claude/commands/*.md` — search for the topic area
2. Check if the relevant skill exists at all
3. If it exists, check if the specific error pattern is covered
4. Classify the gap:
   - **MISSING SKILL**: No skill covers this area → recommend creating one
   - **MISSING RULE**: Skill exists but doesn't cover this specific pattern → patch the skill
   - **BURIED RULE**: Rule exists but is too deep in the file to be effective → move it up
   - **VAGUE RULE**: Rule exists but is too generic to prevent the error → make it specific
   - **STALE RULE**: Rule exists but is outdated → update it
   - **WRONG RULE**: Rule exists but gives incorrect advice → fix it

## Step 4: Draft the Patch

Write the specific addition/change needed:
- For MISSING RULE: write the new rule with a code example showing WRONG vs CORRECT
- For BURIED RULE: identify the line number and propose moving it
- For VAGUE RULE: rewrite with specific, actionable guidance
- For MISSING SKILL: draft a new skill file outline
- For STALE/WRONG: show current vs proposed

Format:

```
## Proposed Patch

**File:** [path]
**Gap type:** [MISSING RULE / BURIED RULE / etc.]
**Error this prevents:** [description]

### Add after line N / Add to section "X":
[the actual content to add]
```

## Step 5: STOP — Show and Wait

Display the proposed patch to the user. Wait for approval before making any changes. Never auto-edit skill files.

## Step 6: Apply the Patch

After user approval:
- Edit the skill file with the patch
- If you keep any kind of cross-session memory or notes file, update it too so the lesson survives beyond this one skill

## Rules

- Always read the FULL target skill file before proposing changes
- Never restructure or rewrite a skill — make minimal, targeted patches
- Place new rules near the TOP of the relevant section (not buried at the bottom)
- Always include a WRONG vs CORRECT code example when adding a syntax/function rule
- If the same error pattern should be in multiple skills, patch ALL of them
- If you keep any notes on past corrections, check whether one already exists for this error before writing a new one

## Trigger Phrases

Invoke this skill when the user says things like:
- "fix that skill"
- "that shouldn't happen again"
- "add that to the skill"
- "the skill should have caught that"
- "update the skill with this"
- "that's a pattern we need to remember"

## Example Flow

User: "the script used a made-up function name again, fix that skill"

→ Step 1: Error = a nonexistent function name was used
→ Step 2: Root cause = syntax error, wrong function name
→ Step 3: Responsible skill = the relevant language/tool skill. Search... Rule NOT found. Gap type: MISSING RULE
→ Step 4: Draft patch: add to Common Gotchas section: "Function X does not exist. Use Y instead."
→ Step 5: Show patch, wait for approval
→ Step 6: Edit the skill file
