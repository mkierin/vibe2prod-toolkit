```
 __     __ ___  ____   _____   ____    ____                    _
 \ \   / /|_ _|| __ ) | ____| |___ \  |  _ \  _ __  ___    __| |
  \ \ / /  | | |  _ \ |  _|     __) | | |_) || '__|/ _ \  / _` |
   \ V /   | | | |_) || |___   / __/  |  __/ | |  | (_) || (_| |
    \_/   |___||____/ |_____| |_____| |_|    |_|   \___/  \__,_|

        v i b e   →   p r o d   ·   by  k i e r i n
   ───────────────────────────────────────────────────────────
     session handoffs · reality-check · debug · permissions
```

# Vibe2Prod — Claude Code Setup Installer

## ⚡ Quick start — paste this ONE line into Claude Code

> Read https://raw.githubusercontent.com/mkierin/vibe2prod-toolkit/master/install.md and act as the installer it describes. Print the banner first, then show the menu.

Claude fetches this installer, prints the banner, reads the tool **index**, and
shows you a menu. It installs only what you pick — nothing runs without your
confirmation.

Repo: **https://github.com/mkierin/vibe2prod-toolkit**

---

## HOW THIS INSTALLER WORKS (read this, it changes what you do)

This toolkit is an **index**, not a bundle. It does not ship copies of the
tools. It ships a `manifest.json` that lists each tool and the **GitHub repo**
it lives in. Each tool repo carries its own `install.json` describing exactly
what to install (commands, hooks, launchers). Your job as the installer is:

> read the manifest → let the user pick tools → for each pick, clone its repo,
> read its `install.json`, and install what it declares.

Adding a tool later means one new line in `manifest.json` — this installer
picks it up automatically. Never hard-code the tool list below; always read it
live from the manifest.

---

## INSTRUCTIONS TO CLAUDE (the model reading this file)

You are acting as an **installer**. Follow these steps exactly. Do not skip the
banner. Do not skip the menu. Do not install anything the user did not select.
Never overwrite an existing file blindly — on any collision, show it and ask.

### Step 0 — ALWAYS print the banner first

Print the banner at the top of this file verbatim in a code block, every time.
Then continue.

### Step 1 — Read the tool index

Fetch the manifest:

```bash
curl -fsSL https://raw.githubusercontent.com/mkierin/vibe2prod-toolkit/master/manifest.json
```

Parse the `tools` array. Each entry has: `id`, `repo`, `branch`, `install`
(path to its install.json inside the repo), and `desc`.

### Step 2 — Show the menu

Use your `AskUserQuestion` tool (multi-select) to ask **"What do you want to
install?"**. Build the options **dynamically from the manifest** — one option
per tool, using its `id` as the label and `desc` as the description. Add one
more option at the end:

- **Permission settings** — a sane Claude Code permission allowlist (see Step 4)

Wait for the answer. Only act on what they picked.

### Step 3 — Install each selected tool

For every tool the user selected, do this (loop):

```bash
# Given TOOL_REPO, TOOL_BRANCH, TOOL_INSTALL from the manifest entry:
TMP="$(mktemp -d)"
git clone --depth 1 --branch "$TOOL_BRANCH" "$TOOL_REPO" "$TMP"
cat "$TMP/$TOOL_INSTALL"   # <- read this tool's install.json
```

Then honor the tool's `install.json`, which has these optional sections:

1. **`commands`** — a list of file paths inside the repo. Copy each into
   `~/.claude/commands/`:
   ```bash
   mkdir -p ~/.claude/commands
   cp "$TMP"/<each commands path> ~/.claude/commands/
   ```
   Before copying, check for name collisions in `~/.claude/commands/`. If any
   exist, list them and ask: overwrite, skip, or back up first.

2. **`hooks`** — has `files` (copy into `~/.claude/hooks/`, then `chmod +x`)
   and `settings` (a list of hook entries to wire into
   `~/.claude/settings.json`). **Copy the files now; collect the `settings`
   entries and apply them all together in Step 3b.**
   ```bash
   mkdir -p ~/.claude/hooks
   cp "$TMP"/<each hooks.files path> ~/.claude/hooks/
   chmod +x ~/.claude/hooks/*.py 2>/dev/null || true
   ```

3. **`skills`** — a list of `{src, dest}` folder copies. Copy the whole
   `src` folder from the repo to `dest` (expand `~`), e.g.:
   ```bash
   mkdir -p ~/.claude/skills
   cp -r "$TMP/<src>" <dest>
   ```
   No settings.json changes needed — Claude Code discovers `SKILL.md`
   automatically. Check for an existing folder at `dest` first; if present,
   ask: overwrite, skip, or back up.

4. **`bin`** — optional launchers. If present and `optional` is true, ASK the
   user first (tell them the `note` — usually WSL + Windows Terminal + zellij
   only). Only if yes, copy each `files[].src` to its `files[].dest` (expand
   `~`), `mkdir -p` the dest dir, and `chmod +x`. If they say no, skip — the
   tool still works.

Then `rm -rf "$TMP"`.

### Step 3b — Wire hook settings (only if any tool declared `hooks.settings`)

Collect every `settings` entry from every installed tool, then merge them into
`~/.claude/settings.json` under `hooks`. Each entry looks like:

```json
{ "event": "PreToolUse", "matcher": "Bash",
  "command": "python3 ~/.claude/hooks/reality-check-gate.py" }
```

Merge rules:
- **Back up first**: copy `~/.claude/settings.json` to
  `~/.claude/settings.json.bak` before writing.
- Under `settings.json` → `hooks` → `<event>` (e.g. `PreToolUse`, `Stop`),
  append a hook group `{ "matcher": <matcher, if any>, "hooks": [ { "type":
  "command", "command": <command> } ] }`.
- **Never clobber.** Preserve every existing hook. **Dedupe**: if a hook with
  the same `command` is already present for that event, skip it.
- Show the user the resulting `hooks` block and confirm before writing.
- Tell them hooks take effect on the next Claude Code start.

### Step 4 — Permission settings (only if the user chose it)

Give the user a working permission allowlist in `~/.claude/settings.json`
without pasting any secrets.

1. Read the existing `~/.claude/settings.json` if present — you will **merge**,
   never clobber. Show the diff before writing.

2. Ask which **profile** (use `AskUserQuestion`, single-select):

   - **Balanced (recommended)** — three-list model: `allow` safe repetitive
     commands (builds, tests, read-only git), `ask` before destructive/outbound
     ones (`git push`, `rm`, publish), `deny` secrets outright (`.env`,
     `~/.ssh`). Fewest prompts without going blind. Sets `defaultMode:
     "acceptEdits"`.
   - **Trusting** — `Bash(*)` + core tools. Fastest, but removes bash from
     safety checks entirely — sandbox/VM only.
   - **Curated** — a hand-picked allowlist, no `ask`/`deny` layer.

**Balanced profile (recommended):**

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

> **`deny` always wins** — rules evaluate deny → ask → allow. Reads/grep/find
> are free by default; only list what would otherwise prompt.

**Trusting profile** — merge into `permissions.allow`:

```json
{
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" },
  "permissions": { "allow": [ "Read", "Glob", "Grep", "Edit", "Write", "WebFetch", "WebSearch", "Bash(*)" ] }
}
```

**Curated profile** — merge into `permissions.allow`:

```json
{
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" },
  "permissions": {
    "allow": [
      "Read", "Glob", "Grep", "Edit", "Write", "WebFetch", "WebSearch",
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
      "Bash(docker-compose logs:*)", "Bash(tmux:*)"
    ]
  }
}
```

**Rules for writing settings:**
- Merge, do not overwrite. Preserve every existing entry. Dedupe each array.
- For Balanced: if the existing `allow` has `Bash(*)`, **warn** — it defeats
  the `ask`/`deny` gates — and offer to remove it.
- **Never** add any entry containing an API key, token, password, or a
  machine-specific absolute path. If the existing file has such an entry, flag
  it as a leaked secret, offer to strip it, tell them to rotate the key, and do
  **not** echo its value.
- Show the final `permissions` block and confirm before saving.

### Step 5 — Finish

Print a short summary: which tools were installed (commands + hooks), whether
launchers/permissions were applied, and remind the user that slash commands are
available immediately, while new hooks/permissions take effect on the next
Claude Code start.
