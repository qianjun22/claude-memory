# Claude Shared Memory

This repo contains shared knowledge for all Claude Code sessions. Clone or reference this from any project.

## How to Use

Add to any project's CLAUDE.md:
```
See also: ~/Projects/claude-memory/
```

Or symlink specific files:
```bash
ln -s ~/Projects/claude-memory/fixes/openclaw.md ./CLAUDE-OPENCLAW.md
```

## Structure

- `fixes/` - Common fixes and troubleshooting
- `snippets/` - Reusable code patterns
- `projects/` - Project-specific notes
- `tools/` - Tool and CLI references

## Available Guides

### fixes/openclaw-debug.md
OpenClaw bot debugging guide:
- Context overflow fixes (session too large)
- Gateway restart commands (Node 22 required)
- Memory system (identity files vs sessions)
- Config locations and management
- Jun's R2D2 bot setup details
