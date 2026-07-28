---
description: Install the claude-pace statusline bundled with this plugin
allowed-tools: Bash, Read, Write, Edit
---

# claude-pace Setup

You are installing claude-pace, a lightweight statusline for Claude Code.
This skill is idempotent: safe to run for both first install and subsequent updates.

It installs the exact `claude-pace.sh` this plugin version ships. It makes no
network calls, so the version the user gets is whatever `/plugin update` last
resolved — not whatever happens to be on the repository's main branch right now.

Follow these steps in order. If any step fails, stop and explain the issue to the user.

## Step 1: Check prerequisites

Run: `command -v jq`

If jq is not found, tell the user to install it (`brew install jq` on macOS, `apt install jq` on Linux) and stop.

## Step 2: Install the bundled script

```bash
mkdir -p ~/.claude
cp "${CLAUDE_PLUGIN_ROOT}/claude-pace.sh" ~/.claude/statusline.sh
chmod +x ~/.claude/statusline.sh
```

If the `cp` fails because the source path does not exist, the plugin installation
itself is broken. Stop and tell the user to reinstall the plugin. Do not fall back
to downloading the script from the network.

## Step 3: Configure statusline

Read `~/.claude/settings.json` with the Read tool. Then use the Edit tool to add or update the `statusLine` key:

```json
"statusLine": {
  "type": "command",
  "command": "~/.claude/statusline.sh"
}
```

If `statusLine` already exists, update the `command` value. If it does not exist, add it as a top-level key.

## Step 4: Confirm

Tell the user:

- claude-pace has been installed (or updated) successfully.
- Restart Claude Code (or start a new session) to see the statusline.
- To update later, pull the new plugin version first, then re-run this command:
  `/plugin marketplace update claude-pace-marketplace` → `/plugin update claude-pace` → `/reload-plugins` → `/claude-pace:setup`
- To remove: delete the `statusLine` block from `~/.claude/settings.json`.
