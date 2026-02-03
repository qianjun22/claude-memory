# OpenClaw Fixes

## Stuck Bot (Session Issues)

**Symptoms:** Bot stops responding. Common causes:
- **Context overflow**: "prompt too large for the model" - session history grew too big
- **Session corruption**: "unexpected tool_use_id" API errors

**Fix:**
```bash
~/Projects/openclaw/fix-stuck.sh
```

Or manually:
```bash
# Backup corrupted sessions
mkdir -p ~/.openclaw/agents/main/sessions/backup
mv ~/.openclaw/agents/main/sessions/*.jsonl ~/.openclaw/agents/main/sessions/backup/

# Clear session index
echo '{}' > ~/.openclaw/agents/main/sessions/sessions.json

# Restart gateway
npx openclaw gateway restart
```

**Causes:**
- **Context overflow**: Session history exceeds model's context window (auto-compaction may fail to keep up)
- **Session corruption**: Conversation history gets out of sync - a tool_result exists without matching tool_use block

## Preventive Settings

Add to `~/.openclaw/openclaw.json` under `agents.defaults`:

```json
"compaction": {
  "mode": "safeguard",
  "reserveTokensFloor": 30000,
  "maxHistoryShare": 0.4
}
```

- `reserveTokensFloor`: Triggers compaction earlier (default 20000)
- `maxHistoryShare`: Limits history to 40% of context (default 0.5)

## Useful Commands

| Command | Description |
|---------|-------------|
| `npx openclaw health` | Quick health check |
| `npx openclaw status` | Full status with channels |
| `npx openclaw doctor` | Run diagnostics |
| `npx openclaw logs` | View logs |
| `npx openclaw gateway restart` | Restart the bot |

## Config Locations

- Main config: `~/.openclaw/openclaw.json`
- Sessions: `~/.openclaw/agents/main/sessions/`
- Workspace: `~/.openclaw/workspace/`
- Logs: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
