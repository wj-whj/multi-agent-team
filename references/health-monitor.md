# Health Monitor

Session 健康自愈机制。Boss 通过 cron 定期检查团队 agent 的 session 状态，并自动执行恢复操作。

## 自愈等级

| Token 占用 | 等级 | 动作 |
|---|---|---|
| <70% | 正常 | 静默，不发消息 |
| 70-85% | 预警 | 群里提醒该 agent 尽快完成当前任务后 /new |
| 85-95% | 警告 | 自动给该 agent 发 /new 指令重置 session |
| >95% | 紧急 | 自动 /new + 告警通知人类，可能需要手动检查 |

## 自动恢复手段

1. **Session 重置** — 通过 `sessions_send` 给目标 agent 发送 /new 指令
2. **Session 文件清理** — 如果 agent 无响应，直接移除过大的 session 文件：
   ```bash
   mv ~/.openclaw/agents/<agent>/sessions/<id>.jsonl ~/.openclaw/agents/<agent>/sessions/<id>.jsonl.bak
   ```
3. **Gateway 重启** — 如果多个 agent 同时异常，执行 `openclaw gateway restart`
4. **状态验证** — 恢复操作后 60 秒再次检查，确认 agent 恢复正常

## Cron 配置

```json
{
  "name": "team-health-check",
  "schedule": { "kind": "every", "everyMs": 1800000 },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "团队健康自愈检查：\n1. 运行 `openclaw status --deep` 查看所有 agent session 状态\n2. 检查每个 agent 的 token 使用率\n3. 自愈策略：\n   - 70-85%：群里 @ 提醒该 agent 尽快 /new\n   - 85-95%：自动给该 agent 发 /new 重置 session\n   - >95%：自动 /new + 通知人类\n4. 如果 agent 无响应，移除其 session 文件并重启 gateway\n5. 恢复操作后 60 秒再次验证\n6. 全部正常则不发消息\n7. 将检查结果记录到 memory/health-log.md"
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

- 定期检查团队 session 状态（cron 每 30 分钟自动执行）
- 70-85% token 占用：群里提醒 agent
- 85-95%：自动重置 session（/new）
- >95%：自动重置 + 通知人类
- agent 无响应时：清理 session 文件 + 重启 gateway
- 所有恢复操作后验证结果
- 健康日志记录在 memory/health-log.md
```

## 手动触发

除了 cron 自动执行，Boss 也可以随时手动检查：

```
@Boss 检查团队健康状态
```
