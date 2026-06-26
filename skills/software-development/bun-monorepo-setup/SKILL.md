---
name: bun-monorepo-setup
description: "Set up a Bun/Node.js monorepo for local development after cloning from GitHub."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [Bun, Monorepo, Setup, Development, Turbo, Node.js]
    related_skills: [github-repo-management, github-auth]
---

# Bun Monorepo Setup

Set up a Bun-based monorepo (Turbo workspaces) for local development. Covers the full workflow from clone to running dev server.

## When to Use

- User asks to install, set up, or self-host an open-source project
- Project uses Bun as package manager (look for `bun.lock`, `packageManager` in package.json)
- Project is a monorepo with Turbo (`turbo.json`, `workspaces` in package.json)
- Typical projects: Next.js apps, Hono APIs, full-stack TypeScript projects

## Prerequisites Check

```bash
# Check if bun is installed (may not be in PATH)
export PATH="$HOME/.bun/bin:$PATH"
bun --version || echo "bun not installed"

# If not installed:
curl -fsSL https://bun.sh/install | bash
```

**Pitfall:** Bun is often installed at `~/.bun/bin/bun` but not in PATH. Always check there first before attempting install.

## Setup Steps

### 1. Clone the Repository

```bash
cd ~/Desktop/pro  # or user's preferred location
git clone https://github.com/OWNER/REPO.git
cd REPO
```

### 2. Install Dependencies

```bash
export PATH="$HOME/.bun/bin:$PATH"
bun install
```

**Pitfall:** Browser extension postinstall scripts often fail (e.g., `supermemory-browser-extension`). This is usually non-blocking — core app functionality works fine. Don't alarm the user about this.

### 3. Find and Configure Environment Variables

Monorepos often have .env files in subdirectories, not root:

```bash
# Search for env example files
find . -name ".env.example" -o -name ".env.local.example" -o -name ".env.template" 2>/dev/null
```

Common locations:
- `apps/web/.env.example` (Next.js web app)
- `apps/api/.env.example` (API server)
- `.env.example` (root, less common in monorepos)

```bash
# Copy example files
cp apps/web/.env.example apps/web/.env.local
# Edit with required values
```

### 4. Find the Right Dev Script

Check `package.json` scripts — monorepos often have multiple dev commands:

```bash
cat package.json | grep -A 20 '"scripts"'
```

Common patterns:
- `bun run dev` — Production-like dev (may need internal tools like portless)
- `bun run dev:local` — OSS contributor mode (plain localhost ports)
- `turbo run dev` — Direct turbo invocation

**Pitfall:** `dev` vs `dev:local` — In projects like supermemory, `dev` routes through internal proxy tools (*.dev.supermemory.ai) while `dev:local` uses plain localhost. Always prefer `dev:local` for user setups unless they need the internal tooling.

### 5. Start Development Server

```bash
export PATH="$HOME/.bun/bin:$PATH"
bun run dev:local  # or bun run dev
```

Typical ports:
- Web app: http://localhost:3000
- API/MCP: http://localhost:8788
- Docs: http://localhost:3003

## Checking Setup Documentation

Always check these files in order:
1. `README.md` — Quick start, overview
2. `CONTRIBUTING.md` — Detailed dev setup, prerequisites, scripts
3. `CLAUDE.md` / `.cursorrules` — AI-specific instructions, often has architecture overview
4. `apps/web/.env.example` — Required environment variables

## Common Issues

| Issue | Solution |
|-------|----------|
| `env: node: No such file or directory` | Bun projects don't need Node.js installed separately — bun has its own runtime |
| Postinstall script errors (browser extensions) | Non-blocking, ignore for core functionality |
| Missing .env variables | Check all `apps/*/.env.example` files, not just root |
| Port conflicts | Check `turbo.json` or individual app configs for port numbers |
| OAuth not working on localhost | Some projects require `dev:local` mode; OAuth may need special proxy setup |

## Monorepo Structure Reference

```
project/
├── apps/
│   ├── web/          # Next.js frontend
│   ├── api/          # Hono/Express API
│   └── mcp/          # MCP server
├── packages/
│   ├── ui/           # Shared UI components
│   ├── lib/          # Shared utilities
│   └── types/        # TypeScript types
├── turbo.json        # Turbo config
├── biome.json        # Linting config
├── bun.lock          # Lock file
└── package.json      # Root with workspaces
```
