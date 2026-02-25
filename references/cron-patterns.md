# Cron Patterns

Common cron task templates for multi-agent teams. Use `cron` tool to create.

## System Health Monitor (Ops)

Hourly check on services. Assign to ops agent.

```json
{
  "name": "ops-health-monitor",
  "schedule": { "kind": "every", "everyMs": 3600000 },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "心跳监控：检查系统运行状态。\n1. 检查关键服务是否正常响应\n2. 检查守护进程状态\n3. 异常则尝试重启并记录\n4. 正常则简要报告"
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "to": "channel:__NOTIFICATION_CHANNEL_ID__"
  }
}
```

## Daily News Digest (Main)

Daily international news summary at 9:00 AM.

```json
{
  "name": "国际热点新闻",
  "schedule": { "kind": "cron", "expr": "0 9 * * *", "tz": "__USER_TIMEZONE__" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "搜索今天的10大国际热点新闻，每条总结成50字，用中文推送"
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "to": "channel:__NOTIFICATION_CHANNEL_ID__"
  }
}
```

## Notification Channel

Recommend creating a dedicated Discord channel (e.g. #通知 or #消息发布) for all cron task output. This keeps automated messages separate from team discussions.

Create via Discord UI or message tool:
```
message action=channel-create channel=discord guildId=__GUILD_ID__ name=通知
```

Then use the new channel ID in all delivery configs.

## Tips

- Use `sessionTarget: "isolated"` for all cron tasks to avoid polluting main session
- Set `delivery.channel: "discord"` and `delivery.to: "channel:<id>"` for Discord delivery
- Each cron run reuses its session — if content hasn't changed, the agent may skip. For fresh runs, the session will eventually be cleaned up
- Use lighter models for cron tasks when possible (e.g. sonnet instead of opus) via `payload.model`
