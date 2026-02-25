# Multi-Agent Team Skill for OpenClaw

一键部署多智能体协作团队的 OpenClaw Skill。

## 什么是这个 Skill？

这是一个 [OpenClaw](https://github.com/openclaw/openclaw) 的 Agent Skill，用于快速搭建一个 3 人 AI 团队：

| 角色 | 代号 | 职责 | 推荐模型 |
|------|------|------|----------|
| 📋 Boss | 1号 | 项目经理，规划、协调、决策 | Claude Opus |
| 💻 Dev | 2号 | 开发工程师，写代码、实现功能 | Claude Sonnet |
| 🔧 Ops | 3号 | 运维工程师，部署、测试、监控 | Claude Sonnet |

三个 Agent 通过 Discord 群聊协作，像真人团队一样讨论、分工、执行任务。

## 特性

- **交互式部署** — Agent 读取 Skill 后引导你一步步配置，无需手动编辑 JSON
- **完整配置模板** — openclaw.json、工作区文件、协作规则全部模板化
- **踩坑记录** — 包含实际部署中遇到的所有问题和解决方案
- **Cron 任务模板** — 预置系统监控、新闻推送等常用定时任务
- **跨平台** — 支持 macOS 和 Linux

## 快速开始

### 前置条件

1. 已安装 [OpenClaw](https://github.com/openclaw/openclaw)
2. 3 个 Discord Bot Token（[创建教程](https://discord.com/developers/applications)）
3. 至少一个 AI 模型 API 端点（Anthropic 兼容）

### 安装

```bash
git clone https://github.com/wj-whj/multi-agent-team.git ~/.openclaw/skills/multi-agent-team
```

### 使用

在 OpenClaw 聊天中说：

> "帮我搭建多智能体团队"

Agent 会自动读取 Skill 并引导你完成：

1. 收集 Discord Bot Token 和 Guild ID
2. 配置模型提供商
3. 创建工作区目录和文件
4. 应用配置并重启 Gateway
5. 验证三方通信

## 文件结构

```
multi-agent-team/
├── SKILL.md                      # Agent 指令（部署步骤 + 踩坑记录）
└── references/
    ├── config-template.md        # openclaw.json 完整模板
    ├── workspace-templates.md    # 各角色工作区文件模板
    └── cron-patterns.md          # 定时任务配置模板
```

## 架构说明

```
┌─────────────────────────────────────────┐
│              Discord Guild              │
│                                         │
│  #项目组群 (团队讨论)                      │
│  ┌──────┐  ┌──────┐  ┌──────┐          │
│  │ Boss │←→│ Dev  │←→│ Ops  │          │
│  │  1号  │  │  2号  │  │  3号  │          │
│  └──┬───┘  └──┬───┘  └──┬───┘          │
│     │         │         │               │
│  #通知 (定时任务推送)                      │
└─────────────────────────────────────────┘
       │         │         │
  ┌────┴────┐ ┌──┴───┐ ┌──┴───┐
  │workspace│ │ws-dev│ │ws-ops│
  └─────────┘ └──────┘ └──────┘
```

- 每个 Agent 有独立的 Discord Bot、工作区和模型配置
- Agent 间通过 @ mention 在群聊中自然对话
- 支持 Agent-to-Agent 直接通信
- 定时任务推送到专用通知频道

## 踩坑记录

以下是实际部署中踩过的坑，已写入 SKILL.md：

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Bot 互相看不到消息 | `allowBots` 未开启 | Discord 配置加 `"allowBots": true` |
| Agent 不响应 @ | guild users 列表不全 | 必须包含人类 + 所有 bot 的 user ID |
| Gateway 启动失败 | 配置字段放错位置 | `compaction` 只能放 `agents.defaults` 下 |
| Cron 投递失败 | 跨渠道被拦截 | 开启 `tools.message.crossContext` |
| model 格式错误 | 用了字符串而非对象 | agents.list 中 model 必须是 `{primary, fallbacks}` |

## 扩展团队

Skill 默认是 3 人团队，但可以轻松扩展。SKILL.md 中包含添加第 4 个 Agent 的完整步骤。

## 许可

MIT
