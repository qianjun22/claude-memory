# OpenClaw Fixes

## Dual Bot Setup (Mirror Failover)

Two bots run on the machine that can fix each other:

| Bot | Profile | Telegram | Port | State Dir | Service |
|-----|---------|----------|------|-----------|---------|
| Bot1 | (default) | @C3poclawbot | 18789 | ~/.openclaw/ | openclaw-gateway.service |
| Bot2 | bot2 | @MC3PObot | 19001 | ~/.openclaw-bot2/ | openclaw-gateway-bot2.service |

### Fix Scripts

- **Fix Bot1** (run from Bot2): `~/Projects/openclaw/fix-bot1.sh`
- **Fix Bot2** (run from Bot1): `~/Projects/openclaw/fix-bot2.sh`

### Bot2 Commands

```bash
npx openclaw --profile bot2 health
npx openclaw --profile bot2 status
npx openclaw --profile bot2 logs
systemctl --user restart openclaw-gateway-bot2.service
```

## Stuck Bot (Session Issues)

**Symptoms:** Bot stops responding. Common causes:
- **Context overflow**: "prompt too large for the model" - session history grew too big
- **Session corruption**: "unexpected tool_use_id" API errors

**Fix:**
```bash
# For Bot1:
~/Projects/openclaw/fix-bot1.sh

# For Bot2:
~/Projects/openclaw/fix-bot2.sh

# Or the generic script (Bot1 only):
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

Both bots have these settings in their config under `agents.defaults`:

```json
"compaction": {
  "mode": "safeguard",
  "reserveTokensFloor": 30000,
  "maxHistoryShare": 0.4
}
```

- `reserveTokensFloor`: Triggers compaction earlier (default 20000)
- `maxHistoryShare`: Limits history to 40% of context (default 0.5)

## Python 3.12 (Conda)

Both bots use Python 3.12 from conda via `tools.exec.pathPrepend`:

```json
"tools": {
  "exec": {
    "pathPrepend": ["/home/dnnbox/miniconda3/envs/bygen/bin"]
  }
}
```

## Useful Commands

### Bot1 (default)

| Command | Description |
|---------|-------------|
| `npx openclaw health` | Quick health check |
| `npx openclaw status` | Full status with channels |
| `npx openclaw doctor` | Run diagnostics |
| `npx openclaw logs` | View logs |
| `npx openclaw gateway restart` | Restart the bot |

### Bot2

| Command | Description |
|---------|-------------|
| `npx openclaw --profile bot2 health` | Quick health check |
| `npx openclaw --profile bot2 status` | Full status with channels |
| `npx openclaw --profile bot2 doctor` | Run diagnostics |
| `npx openclaw --profile bot2 logs` | View logs |
| `systemctl --user restart openclaw-gateway-bot2.service` | Restart the bot |

## Config Locations

### Bot1
- Config: `~/.openclaw/openclaw.json`
- Sessions: `~/.openclaw/agents/main/sessions/`
- Workspace: `~/.openclaw/workspace/`

### Bot2
- Config: `~/.openclaw-bot2/openclaw.json`
- Sessions: `~/.openclaw-bot2/agents/main/sessions/`
- Workspace: `~/.openclaw/workspace-bot2/`

### Shared
- Logs: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- Fix scripts: `~/Projects/openclaw/fix-bot*.sh`
