# Config Template Reference

Complete openclaw.json template for a 3-agent team. Replace all `__PLACEHOLDER__` values.

## Full Config Template

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "my-provider": {
        "baseUrl": "__PLACEHOLDER_BASE_URL__",
        "apiKey": "__PLACEHOLDER_API_KEY__",
        "api": "anthropic-messages",
        "models": [
          {
            "id": "claude-opus-4-6",
            "name": "claude-opus-4-6",
            "reasoning": true,
            "input": ["text"],
            "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
            "id": "claude-sonnet-4-5",
            "name": "claude-sonnet-4-5",
            "reasoning": false,
            "input": ["text"],
            "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "my-provider/claude-sonnet-4-5",
        "fallbacks": []
      },
      "workspace": "~/.openclaw/workspace",
      "compaction": {
        "mode": "default",
        "maxHistoryShare": 0.9
      },
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    },
    "list": [
      {
        "id": "main",
        "model": {
          "primary": "my-provider/claude-opus-4-6",
          "fallbacks": ["my-provider/claude-sonnet-4-5"]
        },
        "groupChat": {
          "mentionPatterns": ["@boss", "@1号"]
        }
      },
      {
        "id": "dev",
        "workspace": "~/.openclaw/workspace-dev",
        "model": {
          "primary": "my-provider/claude-sonnet-4-5",
          "fallbacks": []
        },
        "groupChat": {
          "mentionPatterns": ["@dev", "@2号"]
        }
      },
      {
        "id": "ops",
        "workspace": "~/.openclaw/workspace-ops",
        "model": {
          "primary": "my-provider/claude-sonnet-4-5",
          "fallbacks": []
        },
        "groupChat": {
          "mentionPatterns": ["@ops", "@3号"]
        }
      }
    ]
  },
  "tools": {
    "message": {
      "crossContext": {
        "allowWithinProvider": true,
        "allowAcrossProviders": true
      }
    },
    "agentToAgent": {
      "enabled": true,
      "allow": ["main", "dev", "ops"]
    },
    "exec": {
      "security": "full"
    }
  },
  "bindings": [
    { "agentId": "main", "match": { "channel": "discord", "accountId": "main" } },
    { "agentId": "dev", "match": { "channel": "discord", "accountId": "dev" } },
    { "agentId": "ops", "match": { "channel": "discord", "accountId": "ops" } }
  ],
  "channels": {
    "discord": {
      "allowBots": true,
      "groupPolicy": "open",
      "dmPolicy": "pairing",
      "allowFrom": ["__PLACEHOLDER_HUMAN_DISCORD_ID__"],
      "dm": {
        "enabled": true,
        "policy": "allowlist"
      },
      "guilds": {
        "__PLACEHOLDER_GUILD_ID__": {
          "requireMention": true,
          "users": [
            "__PLACEHOLDER_HUMAN_DISCORD_ID__",
            "__PLACEHOLDER_BOSS_BOT_USER_ID__",
            "__PLACEHOLDER_DEV_BOT_USER_ID__",
            "__PLACEHOLDER_OPS_BOT_USER_ID__"
          ]
        }
      },
      "accounts": {
        "main": { "token": "__PLACEHOLDER_BOSS_BOT_TOKEN__", "groupPolicy": "open" },
        "dev": { "token": "__PLACEHOLDER_DEV_BOT_TOKEN__", "groupPolicy": "open" },
        "ops": { "token": "__PLACEHOLDER_OPS_BOT_TOKEN__", "groupPolicy": "open" }
      }
    }
  },
  "messages": {
    "ackReactionScope": "group-mentions"
  },
  "plugins": {
    "entries": {
      "discord": { "enabled": true }
    }
  }
}
```

## Placeholder Reference

| Placeholder | Description | How to get |
|---|---|---|
| `__PLACEHOLDER_BASE_URL__` | Model API endpoint | Your provider's API URL |
| `__PLACEHOLDER_API_KEY__` | Model API key | Your provider's dashboard |
| `__PLACEHOLDER_HUMAN_DISCORD_ID__` | Your Discord user ID | Discord > Settings > Advanced > Developer Mode, right-click yourself |
| `__PLACEHOLDER_GUILD_ID__` | Discord server ID | Right-click server name > Copy Server ID |
| `__PLACEHOLDER_BOSS_BOT_TOKEN__` | Boss bot token | Discord Developer Portal > Bot > Token |
| `__PLACEHOLDER_BOSS_BOT_USER_ID__` | Boss bot user ID | Bot joins server, right-click bot > Copy User ID |
| `__PLACEHOLDER_DEV_BOT_TOKEN__` | Dev bot token | Same as above for Dev bot |
| `__PLACEHOLDER_DEV_BOT_USER_ID__` | Dev bot user ID | Same as above |
| `__PLACEHOLDER_OPS_BOT_TOKEN__` | Ops bot token | Same as above for Ops bot |
| `__PLACEHOLDER_OPS_BOT_USER_ID__` | Ops bot user ID | Same as above |

## Model Recommendations

| Role | Recommended | Why |
|---|---|---|
| Boss (main) | opus-4-6 or sonnet-4-6 | Needs best reasoning for planning and coordination |
| Dev | sonnet-4-5 or sonnet-4-6 | Good balance of speed and code quality |
| Ops | sonnet-4-5 | Fast, sufficient for monitoring and deployment tasks |

Use sonnet for all three if budget is a concern — the team still works well.
