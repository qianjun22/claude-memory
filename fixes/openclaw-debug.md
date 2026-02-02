# OpenClaw Bot Debugging Guide

## Quick Reference

### Config Location
- Main config: `~/.openclaw/openclaw.json`
- Cron jobs: `~/.openclaw/cron/jobs.json`
- Sessions: `~/.openclaw/agents/main/sessions/*.jsonl`
- Memory DB: `~/.openclaw/memory/main.sqlite`
- Workspace: `~/.openclaw/workspace/`

### Bot Identity Files (in workspace)
- `IDENTITY.md` - Bot name, personality, emoji
- `USER.md` - User info, preferences
- `SOUL.md` - Core behavior guidelines
- `HEARTBEAT.md` - Periodic task instructions
- `memory/*.md` - Daily notes and memories

### Common Issues

#### Context Overflow Error
```
LLM request rejected: input length and max_tokens exceed context limit
```

**Cause:** Session file too large + high max_tokens setting

**Fix:**
1. Clear the session: `rm ~/.openclaw/agents/main/sessions/*.jsonl`
2. Add context management to config (see below)
3. Restart gateway

#### Node Version Error
```
required: { node: '>=22.12.0' }
```

**Fix:** Use nvm to switch to Node 22:
```bash
source ~/.nvm/nvm.sh && nvm use 22
```

### Context Management Config

Add to `~/.openclaw/openclaw.json` under `agents.defaults`:

```json
"contextTokens": 200000,
"contextPruning": {
  "mode": "cache-ttl",
  "ttl": "10m",
  "keepLastAssistants": 3,
  "softTrimRatio": 0.6,
  "hardClearRatio": 0.75,
  "softTrim": {
    "maxChars": 8000,
    "headChars": 2000,
    "tailChars": 2000
  },
  "hardClear": {
    "enabled": true,
    "placeholder": "[Tool output cleared to save context]"
  }
},
"compaction": {
  "mode": "safeguard",
  "maxHistoryShare": 0.4,
  "memoryFlush": {
    "enabled": true,
    "softThresholdTokens": 40000
  }
}
```

To set model maxTokens, add under `models.providers`:
```json
"models": {
  "providers": {
    "anthropic": {
      "baseUrl": "https://api.anthropic.com",
      "models": [
        {
          "id": "claude-sonnet-4-20250514",
          "name": "Claude Sonnet 4",
          "contextWindow": 200000,
          "maxTokens": 16000
        }
      ]
    }
  }
}
```

### Gateway Commands

**Start gateway (Node 22 required):**
```bash
source ~/.nvm/nvm.sh && nvm use 22
nohup npx openclaw gateway run --bind loopback --port 18789 --force > /tmp/openclaw-gateway.log 2>&1 &
```

**Kill gateway:**
```bash
pkill -9 -f "openclaw-gatewa"
```

**Check status:**
```bash
# Port listening
ss -ltnp | grep 18789

# Channel status
npx openclaw channels status --probe

# Gateway logs
tail -f /tmp/openclaw-gateway.log
```

### Memory System

**Bot memory persists in files, not sessions:**
- Sessions = conversation history (can be cleared)
- Memory files = persistent knowledge (survives clears)

**Memory locations:**
- `~/.openclaw/workspace/memory/YYYY-MM-DD.md` - Daily notes
- `~/.openclaw/workspace/IDENTITY.md` - Bot identity
- `~/.openclaw/workspace/USER.md` - User preferences

**Cron jobs persist separately:**
- Stored in `~/.openclaw/cron/jobs.json`
- Survive session clears
- Check with: `cat ~/.openclaw/cron/jobs.json`

### Jun's Setup

- **Bot name:** R2D2
- **Channel:** Telegram (@R2d2clawbot)
- **Timezone:** America/Los_Angeles
- **Daily podcast:** 5:40 AM (generate) → 6:30 AM (alarm)
- **Podcast style:** Two-voice Bloomberg format (Sarah + David)
