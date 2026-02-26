# Task Tracking

结构化任务生命周期追踪，替代纯聊天记录。

## 目录结构

在 Boss workspace 下创建：

```
tasks/
├── active/          # 进行中的任务
├── done/            # 已完成的任务
└── BOARD.md         # 任务看板总览
```

## 任务文件格式

每个任务一个 md 文件，命名：`YYYY-MM-DD-简短描述.md`

```markdown
# 任务：[简短描述]

- 状态：待办 | 进行中 | 待验证 | 完成
- 优先级：高 | 中 | 低
- 负责人：Dev / Ops / Boss
- 创建时间：YYYY-MM-DD HH:mm
- 完成时间：

## 描述

[任务详细描述]

## 进展

- [YYYY-MM-DD HH:mm] 创建任务，分配给 Dev
- [YYYY-MM-DD HH:mm] Dev 开始实现
- [YYYY-MM-DD HH:mm] Dev 完成，交 Ops 验证

## 结果

[最终结果和备注]
```

## BOARD.md 看板格式

```markdown
# 任务看板

## 进行中
- [ ] [任务名] — @负责人 — 创建于 MM-DD

## 待验证
- [ ] [任务名] — @验证人 — 等待验证

## 今日完成
- [x] [任务名] — 完成于 HH:mm
```

## Boss AGENTS.md 中添加的规则

```markdown
## 任务追踪

- 派任务时在 `tasks/active/` 创建任务文件
- 任务完成后移到 `tasks/done/`，更新 BOARD.md
- 每天结束时更新看板，清理已完成任务
- 复杂任务拆分成子任务，每个子任务单独文件
```

## 协作流程

1. Boss 派任务 → 创建任务文件 + 群里 @ 负责人
2. 负责人确认 → 更新状态为"进行中"
3. 完成后 → 群里汇报 + 更新状态为"待验证"
4. 验证通过 → 移到 done/ + 更新看板
