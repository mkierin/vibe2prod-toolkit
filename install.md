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

   - **Trusting** — broad access, fewest prompts. Adds `Bash(*)`,
     `WebFetch`, `WebSearch`, and the core file tools. Best for a solo
     dev on their own machine who trusts their own commands.
   - **Curated** — explicit allowlist of common safe commands, still
     prompts for anything unusual. Best if you want a tighter setup.

3. Merge the chosen block into `permissions.allow` (dedupe against what's
   already there) and set the env var. Write the file. Then print what
   changed.

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
- Deduplicate the `allow` array.
- **Never** add any entry that contains an API key, token, password, or a
  machine-specific absolute path. If the user's existing file contains such
  entries, leave them untouched but do **not** echo their values back.
- Show the user the final `permissions.allow` list and confirm before saving.

---

### Step 4 — Finish

Print a short summary of what was installed (skills copied, permission
profile applied) and remind the user they can start using the slash
commands immediately. If the user restarts Claude Code, the new
permissions take effect for the whole session.
