# Health Monitor

Session 健康自愈机制。Boss 通过 cron 定期检查团队 agent 的 session 状态。

## 原理

- 每 30 分钟检查一次各 agent session 的 token 使用率
- token 占用 >85% → 在群里提醒该 agent 需要 /new
- token 占用 >95% → 直接告警，建议人工介入

## Cron 配置

```json
{
  "name": "team-health-check",
  "schedule": { "kind": "every", "everyMs": 1800000 },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "团队健康检查：\n1. 运行 `openclaw status --deep` 查看所有 agent session 状态\n2. 检查每个 agent 的 token 使用率\n3. 如果任何 agent 的 session token 占用超过 85%，在 Discord 群里 @ 提醒该 agent 需要 /new 开新 session\n4. 如果超过 95%，标记为紧急，建议人工介入\n5. 如果全部正常，不需要发消息\n6. 将检查结果记录到 memory/health-log.md"
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "to": "channel:__NOTIFICATION_CHANNEL_ID__"
  }
}
```

## Boss AGENTS.md 中添加的规则

```markdown
## 健康自愈

- 定期检查团队 session 状态（cron 自动执行）
- 发现 token 占用 >85% 时在群里提醒对应 agent
- 发现 token 占用 >95% 时告警并建议人工介入
- 健康日志记录在 memory/health-log.md
```
