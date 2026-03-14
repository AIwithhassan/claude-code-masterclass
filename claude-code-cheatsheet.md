# Claude Code — Key Concepts Guide

A simple reference explaining five core Claude Code features with plain-language descriptions and practical examples.

---

## 🎬 Video Walkthrough

Watch the full explanation here: [Click Here](https://www.youtube.com/watch?v=U2lHEAN21KY)

---

## 🔗 Useful Resources

| Resource | Link |
|---|---|
| 🤖 **What Models Can I Run?** — Check which AI models your hardware can handle | [whatmodelscanirun.com](https://whatmodelscanirun.com/?gpu=m2-pro&mem=1&ctx=64) |
| 🦙 **Ollama** — Run large language models locally on your machine | [ollama.com](https://ollama.com) |
| 🔀 **OpenRouter** — Access hundreds of AI models through a single API | [openrouter.ai](https://openrouter.ai) |

---

## Table of Contents

- [Skills](#-skills)
- [Raw MCP](#-raw-mcp)
- [Plugins](#-plugins)
- [Remote Control](#-remote-control)
- [Hooks](#-hooks)

---

## 🧠 Skills

**What it is:** A custom instruction file (Markdown) that teaches Claude a new workflow or capability. Write it once, and Claude can use it automatically or you can call it with `/skill-name`.

**Think of it as:** A recipe card you hand Claude. It follows the recipe when you ask.

### Create a Skill

Create `~/.claude/skills/fix-issue/SKILL.md`:

```markdown
---
name: fix-issue
description: Fix a GitHub issue by number. Use when the user asks to fix an issue.
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS:
1. Read the issue description
2. Find relevant files
3. Implement the fix
4. Write tests
5. Commit the changes
```

### Use it

```bash
/fix-issue 42
```

Claude reads issue #42 and fixes it step by step.

### Bundled Skills (built into Claude Code)

| Skill | What it does |
|---|---|
| `/simplify` | Reviews recent changes for quality/efficiency issues and fixes them |
| `/batch <instruction>` | Orchestrates large-scale parallel changes across a codebase |
| `/debug` | Troubleshoots your current Claude Code session |
| `/loop [interval] <prompt>` | Runs a prompt repeatedly on a schedule |

### Skill Frontmatter Options

```markdown
---
name: my-skill
description: What it does and when to use it
disable-model-invocation: true   # only YOU can invoke it (not Claude automatically)
user-invocable: false            # only Claude can invoke it (hidden from / menu)
allowed-tools: Read, Grep, Glob  # restrict which tools Claude can use
context: fork                    # run in an isolated subagent
---
```

### Where Skills Live

| Location | Scope |
|---|---|
| `~/.claude/skills/<name>/SKILL.md` | All your projects (personal) |
| `.claude/skills/<name>/SKILL.md` | This project only |
| `<plugin>/skills/<name>/SKILL.md` | Where the plugin is enabled |

---

## 🔌 Raw MCP

**What it is:** Running Claude Code itself *as* an MCP server — so other apps (Claude Desktop, custom scripts, other AI agents) can connect to Claude Code and use its tools (read files, edit files, run Bash, etc.) over the Model Context Protocol.

**Think of it as:** Turning Claude Code inside-out. Instead of Claude Code *using* MCP tools, it *becomes* an MCP server that other tools use.

### Start Claude Code as an MCP Server

```bash
claude mcp serve
```

### Connect from Claude Desktop

Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "claude",
      "args": ["mcp", "serve"]
    }
  }
}
```

Now Claude Desktop can ask the `claude-code` server to read files, edit code, run commands, and more using your local environment.

### When to Use Raw MCP

- Building a custom app or agent that needs to drive Claude Code programmatically
- Exposing Claude Code's local tools to another AI tool
- Automating Claude Code operations from scripts or CI pipelines

---

## 📦 Plugins

**What it is:** A packaged, shareable bundle of Skills + Agents + Hooks + MCP servers in a single folder. Unlike standalone `.claude/` configs (per-project only), plugins can be distributed to teammates or a marketplace.

**Think of it as:** An "app" for Claude Code — install it once, get new capabilities everywhere.

### Plugin Structure

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json        ← identity & metadata (required)
├── skills/
│   └── code-review/
│       └── SKILL.md       ← packaged skills
├── agents/                ← custom subagents
├── hooks/
│   └── hooks.json         ← event hooks
└── .mcp.json              ← bundled MCP servers
```

### plugin.json

```json
{
  "name": "my-plugin",
  "description": "Code review and quality tools",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

### Test Locally

```bash
claude --plugin-dir ./my-plugin
/my-plugin:code-review src/auth.ts
```

### Plugins vs Standalone Configuration

| Feature | Standalone `.claude/` | Plugin |
|---|---|---|
| Scope | One project only | Shareable across projects |
| Skill name | `/deploy` | `/my-plugin:deploy` (namespaced) |
| Best for | Quick experiments | Versioned, distributable tools |
| Distribution | Manual copy | Install via marketplace |

### Install an Existing Plugin

```bash
/plugin install <marketplace-url-or-path>
```

---

## 📱 Remote Control

**What it is:** A way to continue your local Claude Code session from any other device (phone, tablet, another browser) — without moving anything to the cloud. Claude keeps running on *your machine*; the remote device is just a window into it.

**Think of it as:** Remote desktop, but just for your Claude Code terminal session.

### Start a Remote Session

```bash
cd my-project
claude remote-control --name "My Feature Work"
```

Claude prints a session URL and QR code. Scan from your phone or open in another browser → you're now controlling the same local session.

### Connect from Another Device

| Method | How |
|---|---|
| Browser | Open the session URL at [claude.ai/code](https://claude.ai/code) |
| Mobile (QR) | Scan the QR code shown in the terminal |
| Session list | Open claude.ai/code and find the session by name |

### Always-On Mode

```bash
# Inside Claude Code, run /config and set:
Enable Remote Control for all sessions → true
```

### Why Use Remote Control

- ✅ Start coding on your laptop, continue on your phone
- ✅ Your local filesystem, `.env`, MCP servers, and tools stay accessible
- ✅ Session reconnects automatically if your laptop sleeps
- ✅ Multiple devices can view the same conversation simultaneously

### Remote Control vs Claude Code on the Web

| | Remote Control | Claude Code on the Web |
|---|---|---|
| Where it runs | **Your machine** | Anthropic cloud |
| Local files | ✅ Accessible | ❌ Not accessible |
| Local MCP servers | ✅ Available | ❌ Not available |
| Best for | Continuing local work remotely | Starting fresh tasks anywhere |

> **Requirements:** Claude Code v2.1.51+, Pro/Max/Team/Enterprise plan, authenticated with claude.ai.

---

## 🪝 Hooks

**What it is:** Scripts (shell, HTTP endpoint, or AI prompt) that run automatically at specific moments in Claude's lifecycle — before/after a tool call, when a session starts, when Claude finishes, etc.

**Think of it as:** "If Claude does X, automatically run my script Y."

### Hook Events

| Event | When it fires | Can Block? |
|---|---|---|
| `PreToolUse` | Before a tool runs | ✅ Yes |
| `PostToolUse` | After a tool succeeds | ❌ No (already ran) |
| `UserPromptSubmit` | Before Claude processes your message | ✅ Yes |
| `Stop` | When Claude finishes responding | ✅ Yes (force continue) |
| `SessionStart` | When a session begins | ❌ No |
| `SessionEnd` | When a session ends | ❌ No |
| `PermissionRequest` | When a permission dialog appears | ✅ Yes |

### Exit Code Convention

| Exit code | Meaning |
|---|---|
| `0` | ✅ Allow / success |
| `2` | 🚫 Block the action (stderr message shown to Claude) |
| anything else | ⚠️ Non-blocking error (logged only) |

### Example 1 — Block `rm -rf` Commands

`.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": ".claude/hooks/block-rm.sh"
      }]
    }]
  }
}
```

`.claude/hooks/block-rm.sh`:

```bash
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  echo "Destructive command blocked by hook" >&2
  exit 2   # blocks the tool call
fi

exit 0     # allow the command
```

### Example 2 — Auto-Lint After Every File Edit (Async)

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/lint.sh",
        "async": true
      }]
    }]
  }
}
```

→ Every time Claude writes or edits a file, `lint.sh` runs automatically in the background without blocking Claude.

### Example 3 — Prompt-Based Hook (AI-evaluated)

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Evaluate if Claude should stop. Check if all tasks are complete: $ARGUMENTS. Return {\"ok\": true} to allow stopping or {\"ok\": false, \"reason\": \"...\"}  to continue."
      }]
    }]
  }
}
```

### Hook Locations

| File | Scope | Shared? |
|---|---|---|
| `~/.claude/settings.json` | All your projects | No (local only) |
| `.claude/settings.json` | This project | Yes (can commit to repo) |
| `.claude/settings.local.json` | This project | No (gitignored) |
| Plugin `hooks/hooks.json` | When plugin is enabled | Yes (bundled with plugin) |

### Manage Hooks Interactively

Type `/hooks` inside Claude Code to view, add, and remove hooks without editing JSON files directly.

---

## Quick Reference

| Feature | File/Command | Purpose |
|---|---|---|
| Skills | `.claude/skills/<name>/SKILL.md` | Teach Claude custom workflows |
| Raw MCP | `claude mcp serve` | Expose Claude Code's tools to other apps |
| Plugins | `.claude-plugin/plugin.json` | Package and share extensions |
| Remote Control | `claude remote-control` | Access your local session from any device |
| Hooks | `.claude/settings.json` → `hooks` key | Automate actions at lifecycle events |

---

## Further Reading

- [Skills documentation](https://code.claude.com/docs/en/skills)
- [MCP documentation](https://code.claude.com/docs/en/mcp)
- [Plugins documentation](https://code.claude.com/docs/en/plugins)
- [Remote Control documentation](https://code.claude.com/docs/en/remote-control)
- [Hooks reference](https://code.claude.com/docs/en/hooks)
