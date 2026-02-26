---
name: multi-agent-team
description: Deploy a multi-agent team (Boss/Dev/Ops) on OpenClaw with Discord group chat collaboration. Use when setting up a new multi-agent team, configuring agent-to-agent communication, creating Discord-based collaboration workflows, or migrating an existing team setup to a new server. Triggers on requests like "搭建多智能体团队", "setup multi-agent team", "deploy team", "配置团队协作".
---

# Multi-Agent Team Deployment

Deploy a 3-agent team (Boss/Dev/Ops) with Discord group chat collaboration on OpenClaw.

## Architecture

- 📋 Boss (main): Project manager, planning, coordination, final decisions
- 💻 Dev: Developer, writes code, implements features
- 🔧 Ops: Operations, deployment, testing, monitoring

Each agent has its own Discord bot, workspace, and model config. They communicate via @ mentions in a shared Discord channel.

## Prerequisites

Collect from user before starting:

1. **3 Discord bot tokens** (one per agent — create via https://discord.com/developers/applications)
2. **Discord Guild ID** (server where bots will operate)
3. **Model provider** — baseUrl + apiKey (at least one Anthropic-compatible endpoint)
4. **Channel ID** for team chat (or create one during setup)

Each Discord bot must be invited to the guild with these permissions: Send Messages, Read Message History, Add Reactions, Use External Emojis. Use OAuth2 URL with `bot` scope and `permissions=68672`.

## Deployment Steps

### Step 1: Validate environment

```bash
openclaw status
```

Confirm OpenClaw is installed and gateway is running.

### Step 2: Create workspaces

```bash
mkdir -p ~/.openclaw/workspace-dev
mkdir -p ~/.openclaw/workspace-ops
```

Boss uses the default `~/.openclaw/workspace`.

### Step 3: Generate config

Read `references/config-template.md` for the full openclaw.json template.

Apply config using `gateway config.patch`. Key sections to patch:

1. `models.providers` — add user's model provider(s)
2. `agents` — defaults + list of 3 agents
3. `channels.discord` — guild config with all 4 user IDs (human + 3 bots)
4. `bindings` — map each agent to its Discord account
5. `tools.agentToAgent` — enable inter-agent communication
6. `tools.message.crossContext` — enable cross-channel delivery for cron tasks

### Step 4: Write workspace files

For each workspace, create AGENTS.md and SOUL.md. Read `references/workspace-templates.md` for templates.

Key rules to include in Boss's AGENTS.md:
- Discord collaboration rules (natural discussion, not robotic reports)
- Team member Discord tags for @ mentions
- When to delegate vs handle directly

### Step 5: Restart and verify

```bash
openclaw gateway restart
```

Then verify in Discord:
1. Boss @ Dev → Dev responds ✅
2. Boss @ Ops → Ops responds ✅
3. Dev @ Ops → Ops responds ✅

### Step 6: Optional — Set up cron tasks

Common cron patterns for teams. Read `references/cron-patterns.md` for templates.

## Critical Configuration Notes (踩坑记录)

These are hard-won lessons. Do NOT skip:

1. **allowBots: true** — Required in discord config, otherwise bots can't see each other's messages
2. **guild users list** — Must include ALL 4 IDs: human + 3 bot user IDs (not application IDs)
3. **requireMention: true** — Prevents agents from responding to every message
4. **compaction** — Only valid under `agents.defaults`, NOT in individual agent objects in `agents.list`
5. **crossContext** — Enable `tools.message.crossContext` if cron tasks need to deliver to channels other than the originating one
6. **model.primary format** — Use `provider/model-id` format (e.g. `wj-1/claude-opus-4-6`)
7. **model as object** — In agents.list, model must be `{ "primary": "...", "fallbacks": [...] }`, not a plain string
8. **Bot user ID vs Application ID** — The guild users list needs the bot's USER ID (visible when the bot joins the server), not the Application ID from the developer portal
9. **Context window matters** — Use 200k ctx models (e.g. sonnet-4-5) for Dev/Ops; 128k models hit limits fast in long sessions
10. **Session token overflow** — The #1 cause of agent going unresponsive. Set up health monitor cron (see references/health-monitor.md) and proactively /new before hitting limits

## Step 7: Set up task tracking (recommended)

Create task tracking structure in Boss workspace:

```bash
mkdir -p ~/.openclaw/workspace/tasks/active
mkdir -p ~/.openclaw/workspace/tasks/done
```

Create `tasks/BOARD.md` as the task dashboard. Read `references/task-tracking.md` for format and workflow.

### Step 8: Set up health monitor (recommended)

Create a cron job for Boss to periodically check team session health. Read `references/health-monitor.md` for the cron template.

This prevents session token overflow — the #1 cause of agent going unresponsive.

### Step 9: Set up shared knowledge base (optional)

If your agents already share a project directory (e.g. all agents access `~/.openclaw/workspace/projects/`), you can skip this step.

Otherwise, create a shared directory with symlinks so each agent can access it from their own workspace:

```bash
mkdir -p ~/.openclaw/workspace/shared
ln -s ~/.openclaw/workspace/shared ~/.openclaw/workspace-dev/shared
ln -s ~/.openclaw/workspace/shared ~/.openclaw/workspace-ops/shared
```

Use `shared/` for project docs, API specs, architecture decisions, and anything all agents need to reference.

## Extending the Team

To add a 4th agent (e.g. QA):

1. Create a new Discord bot, get token and user ID
2. Add to `agents.list` with id, model, workspace, groupChat.mentionPatterns
3. Add to `channels.discord.accounts` with token
4. Add to `channels.discord.guilds.<id>.users`
5. Add a new binding in `bindings`
6. Add to `tools.agentToAgent.allow`
7. Create workspace directory and files
8. Restart gateway
