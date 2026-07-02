```
 __     __ ___  ____   _____   ____    ____                    _
 \ \   / /|_ _|| __ ) | ____| |___ \  |  _ \  _ __  ___    __| |
  \ \ / /  | | |  _ \ |  _|     __) | | |_) || '__|/ _ \  / _` |
   \ V /   | | | |_) || |___   / __/  |  __/ | |  | (_) || (_| |
    \_/   |___||____/ |_____| |_____| |_|    |_|   \___/  \__,_|

        v i b e   →   p r o d   ·   claude code toolkit
   ───────────────────────────────────────────────────────────
        session handoffs · parallel tabs · debug · wrapup
```

# Vibe2Prod — Claude Code Setup Installer

> Paste **this entire file** into a Claude Code session (or run
> `claude` and paste it). Claude will show you a menu and install only
> what you pick. Nothing runs without your confirmation.

Repo: **https://github.com/mkierin/vibe2prod-toolkit**

---

## INSTRUCTIONS TO CLAUDE (the model reading this file)

You are acting as an **installer**. Follow these steps exactly. Do not
skip the menu. Do not install anything the user did not select.

### Step 1 — Show the menu

Use your `AskUserQuestion` tool (multi-select) to ask **"What do you want
to install?"** with these options:

1. **Skills** — the Vibe2Prod session-workflow command skills
2. **Permission settings** — a sane Claude Code permission allowlist
3. **Both**

Wait for the answer before doing anything.

---

### Step 2 — If the user chose Skills

The skills live in a GitHub repo. Install them like this:

```bash
# 1. The repo
REPO_URL="https://github.com/mkierin/vibe2prod-toolkit.git"

# 2. Clone to a temp dir
TMP="$(mktemp -d)"
git clone --depth 1 "$REPO_URL" "$TMP"

# 3. Copy the command skills into the user's Claude commands dir
mkdir -p ~/.claude/commands
cp "$TMP"/commands/*.md ~/.claude/commands/

# 4. List what was installed
ls ~/.claude/commands/
```

(Keep `$TMP` until after the optional launcher step below, then
`rm -rf "$TMP"` to clean up.)

**Optional launcher (`bin/`):** the repo also ships a `new-session` launcher +
`team-status.sh` that let `/handoff` auto-spawn a fresh tab. It needs **WSL +
Windows Terminal + zellij** and is optional — the skills write handoff notes
fine without it. Ask the user if they want it; only if yes:

```bash
mkdir -p ~/.local/bin
cp "$TMP"/bin/new-session "$TMP"/bin/team-status.sh ~/.local/bin/ 2>/dev/null
chmod +x ~/.local/bin/new-session ~/.local/bin/team-status.sh
# ensure ~/.local/bin is on PATH (add to ~/.profile if missing)
```

Notes for Claude:
- The repo is public — no auth needed to clone.
- **Never overwrite blindly.** Before copying, check whether any target
  filenames already exist in `~/.claude/commands/`. If they do, list the
  collisions and ask the user whether to overwrite, skip, or back up first.
- After copying, tell the user the skills are available as slash commands
  (e.g. `/handoff`, `/wrapup`, `/pickup`).

**The Vibe2Prod skills (what the user is getting):**

| Command | What it does |
|---|---|
| `/handoff` | Write a session handoff note so the next session resumes the right thread |
| `/handoff-many` | Split the session into N parallel handoff notes + spawn a tab per task |
| `/wrapup` | End-of-session close-out: update tasks, commit, write handoff, print a receipt |
| `/pickup` | Morning restore — read the handoff + project state, recommend a first action |
| `/new-session` | Open a fresh Claude session in a new terminal tab |
| `/debug` | Reproduce → isolate layer → 3 hypotheses → fix → regression test. Never skips steps |
| `/postmortem` | Root-cause an incident, then turn each cause into a durable automated check |
| `/fix-skill` | Repair or improve an existing skill after it misfires |

---

### Step 3 — If the user chose Permission settings

The goal is to give the user a working permission allowlist in
`~/.claude/settings.json` **without pasting any secrets**. Do this:

1. Read the user's existing `~/.claude/settings.json` if it exists. If it
   does, you will **merge** — never clobber their current file. Show them
   the diff before writing.

2. Ask which **profile** they want (use `AskUserQuestion`, single-select):

   - **Balanced (recommended)** — the three-list model: `allow` the safe,
     repetitive stuff (builds, tests, read-only git), `ask` before anything
     destructive or outbound (`git push`, `rm`, publish), and `deny` secrets
     outright (`.env`, `~/.ssh`). Fewest prompts *without* going blind. Also
     sets `defaultMode: "acceptEdits"` so file edits don't nag. Best default
     for almost everyone.
   - **Trusting** — broad access, fewest prompts. Adds `Bash(*)`,
     `WebFetch`, `WebSearch`, and the core file tools. Fastest, but `Bash(*)`
     removes bash from safety-checking entirely — only for a solo dev on a
     machine they fully trust (ideally a sandbox/VM).
   - **Curated** — explicit allowlist of common safe commands, still
     prompts for anything unusual. Tighter than Trusting, but has no `ask`
     or `deny` layer — prefer Balanced unless you want to hand-pick.

3. Merge the chosen block into the matching `permissions` lists (`allow`,
   `ask`, `deny` — dedupe each against what's already there), set the env var
   and `defaultMode`, then write the file and print what changed.

> **Note — `deny` always wins.** Rules are evaluated deny → ask → allow, so
> a `deny` entry silently overrides any `allow`/`ask` that matches the same
> thing. That's the point: it makes secrets unreadable no matter what else
> is allowed. Reads/grep/find are already free by default — don't allowlist
> them, only list what would otherwise prompt.

**Balanced profile (recommended)** — merge `allow`/`ask`/`deny` and set
`defaultMode`:

```json
{
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" },
  "defaultMode": "acceptEdits",
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep", "Edit", "Write",
      "WebFetch", "WebSearch",
      "Bash(npm run build)", "Bash(npm run test:*)", "Bash(npm run lint:*)",
      "Bash(git status)", "Bash(git diff:*)", "Bash(git log:*)",
      "Bash(git add:*)", "Bash(git commit:*)",
      "Bash(ls:*)", "Bash(cat:*)", "Bash(find:*)", "Bash(mkdir:*)",
      "Bash(mv:*)", "Bash(cp:*)", "Bash(test:*)"
    ],
    "ask": [
      "Bash(git push:*)", "Bash(rm:*)", "Bash(npm publish:*)",
      "Bash(git reset --hard:*)", "Bash(docker:*)"
    ],
    "deny": [
      "Read(.env)", "Read(./.env.*)", "Read(**/.env)",
      "Read(~/.ssh/**)", "Read(~/.aws/**)",
      "Read(**/*secret*)", "Read(**/*credentials*)"
    ]
  }
}
```

> Never allowlist `Bash(*)` alongside these — it re-opens everything the
> `ask`/`deny` lists are meant to gate.

**Trusting profile** — merge these into `permissions.allow`:

```json
{
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" },
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep", "Edit", "Write",
      "WebFetch", "WebSearch",
      "Bash(*)"
    ]
  }
}
```

**Curated profile** — merge these into `permissions.allow` instead:

```json
{
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" },
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep", "Edit", "Write",
      "WebFetch", "WebSearch",
      "Bash(ls:*)", "Bash(cat:*)", "Bash(grep:*)", "Bash(find:*)",
      "Bash(mkdir:*)", "Bash(mv:*)", "Bash(cp:*)", "Bash(test:*)",
      "Bash(chmod:*)", "Bash(wc:*)", "Bash(sort:*)", "Bash(tree:*)",
      "Bash(env:*)", "Bash(printf:*)", "Bash(tee:*)",
      "Bash(python3:*)", "Bash(python:*)", "Bash(pip install:*)",
      "Bash(pip3 install:*)", "Bash(node:*)", "Bash(npm install:*)",
      "Bash(npm run build:*)", "Bash(curl:*)",
      "Bash(git add:*)", "Bash(git commit:*)", "Bash(git config:*)",
      "Bash(git init:*)", "Bash(git reset:*)",
      "Bash(docker exec:*)", "Bash(docker logs:*)",
      "Bash(docker-compose up:*)", "Bash(docker-compose down:*)",
      "Bash(docker-compose logs:*)",
      "Bash(tmux:*)"
    ]
  }
}
```

**Rules for Claude when writing settings:**
- Merge, do not overwrite. Preserve every entry already in the user's file.
- Deduplicate each of the `allow`, `ask`, and `deny` arrays.
- For Balanced: if the user's existing `allow` already contains `Bash(*)`,
  **warn them** — it defeats the `ask`/`deny` gates — and offer to remove it.
- **Never** add any entry that contains an API key, token, password, or a
  machine-specific absolute path. If the user's existing file contains such
  entries (e.g. a `Bash(...API_KEY=...)` allow rule), **flag it as a leaked
  secret**, offer to strip it, tell them to rotate the key, and do **not**
  echo its value back.
- Show the user the final `permissions` block and confirm before saving.
- **Optional — mention Auto Mode:** on Max/Team/Enterprise you can also set
  `"defaultMode": "auto"`, which uses a classifier to auto-approve low-risk
  actions instead of prefix rules — the safe middle ground between manual
  approval and `bypassPermissions`.

---

### Step 4 — Finish

Print a short summary of what was installed (skills copied, permission
profile applied) and remind the user they can start using the slash
commands immediately. If the user restarts Claude Code, the new
permissions take effect for the whole session.
