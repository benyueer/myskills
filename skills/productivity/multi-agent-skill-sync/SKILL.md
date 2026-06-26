---
name: multi-agent-skill-sync
description: "Sync skills/commands across multiple AI coding agents (Claude Code, Codex, OpenCode, Hermes, Cursor, Windsurf). Covers unified directory strategies, community tools (skillshare, skills-link, cc-switch), and cross-device git sync."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [skills, sync, multi-agent, claude-code, codex, opencode, cursor, cross-device]
    related_skills: [hermes-agent, hermes-agent-skill-authoring]
---

# Multi-Agent Skill Sync

## Overview

When using multiple AI coding agents (Claude Code, Codex, OpenCode, Hermes, Cursor, Windsurf, etc.), each tool stores skills/commands in its own directory with its own format. This creates a "one source, many targets" management problem. This skill covers how to set up a single source of truth and sync across all tools and devices.

## When to Use

- User wants to share skills across Claude Code, Codex, OpenCode, Hermes, Cursor, etc.
- User needs skills synced across multiple machines
- User asks about unified AI agent prompt/instruction management
- User is setting up a new development machine and wants skills available everywhere

## The Problem

Each tool has its own skills directory and format:

| Tool       | Directory                  | Format                     |
|------------|---------------------------|----------------------------|
| Hermes     | ~/.hermes/skills/          | SKILL.md (YAML frontmatter)|
| Claude Code| ~/.claude/commands/        | Markdown slash commands    |
| Codex      | ~/.codex/skills/           | Markdown                   |
| OpenCode   | ~/.config/opencode/        | JSONC config               |
| Cursor     | ~/.cursor/ or project .cursorrules | Rules file          |
| Windsurf   | ~/.windsurf/ or project rules | Rules file               |

## Recommended Approach: Use skillshare

**skillshare** (runkids/skillshare) is the most mature community solution (1700+ stars, actively maintained).

```bash
# Install via brew
brew install farion1231/ccswitch/tap/skillshare

# Or download binary from https://github.com/runkids/skillshare/releases
# macOS arm64: skillshare_X.Y.Z_darwin_arm64.tar.gz
```

How it works:
- Single directory `~/.config/skillshare/skills/` as source of truth
- `skillshare sync` distributes to all detected tools
- Supports 60+ AI coding agents
- Two modes: symlink (default, instant sync) or copy (safer)
- Built-in security audit for prompt injection
- Can install skills from GitHub repos: `skillshare add owner/repo`

```bash
# Quick start
skillshare              # detect apps, import existing skills, create links
skillshare sync         # sync all targets
skillshare add vercel-labs/agent-skills  # install from GitHub
```

## Alternative Tools

### skills-link (npm, symlink-based)
```bash
npm i -g skills-link
skills-link             # symlink all app skills to ~/AISkills/
skills-link add owner/repo -s skill-name  # install from GitHub
```
Approach: each tool's `~/.xxx/skills` becomes a symlink to one master directory.

### cc-switch (GUI, macOS)
- Desktop app with system tray
- SQLite database for config management
- Manages Claude Code, Codex, Gemini, OpenCode
- Skill repos stored in `skill_repos` table
- Auto-sync via `skillSyncMethod: "auto"` in ~/.cc-switch/settings.json

### skrills (Rust, lightweight)
- Supports Claude Code, Codex, Copilot
- Validates and analyzes skills before sync

## DIY: Custom Git Repo + Sync Script

If community tools don't fit, build your own:

```
~/ai-skills/              ← Git repo (single source of truth)
├── hermes/
│   └── my-skill/SKILL.md
├── claude/
│   └── my-skill.md
├── codex/
│   └── my-skill.md
└── sync.sh              ← push/pull/status commands
```

Sync strategy:
- **symlink**: each tool directory links back to repo (instant, but some tools may not follow symlinks)
- **copy**: script copies files to each directory (safer, needs explicit sync)
- Bidirectional: use mtime tracking (.sync-state.json) to detect which side changed
- Conflict: if both sides changed, prompt user to resolve

Cross-device: `git pull` / `git push` on the repo directory.

## Pitfalls

- Some tools modify skills directories on their own (e.g., Claude's learned skills) — don't symlink those
- Symlinks may not work on Windows without admin privileges
- cc-switch only manages Claude/Codex/Gemini/OpenCode — doesn't cover Hermes
- Security scans may block skill installation from community repos (`--force` to override)
- Skill changes may need tool restart to take effect (Hermes: `/reset`, Claude: new session)
