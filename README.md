# OpenClaw Telegram 自愈通知 Skill / OpenClaw Telegram Self-Heal Notification Skill

把 **OpenClaw Telegram 发送失败自愈、4 类卡住判断、窗口归因、恢复后通知** 整理成一份可复用、可发布的 AgentSkill。

这份仓库主要解决的是：

- Telegram 私聊或群窗口“像卡住”
- gateway 还活着，但 `sendMessage` / `sendChatAction` 失败
- 单纯重启 gateway 还不够，还需要知道**影响的是哪个窗口**
- 不是所有卡住都是 Telegram 故障，还可能是：
  - 长任务占住
  - 上下文 / token 过重
  - 路由 / 绑定问题
- 恢复后希望把通知回到**对应主窗口 / 对应群**

## 核心能力

- 检测 Telegram 发送异常
- 自动重启 OpenClaw gateway
- 记录结构化恢复事件
- 对异常进行窗口归因
- 增加 4 类卡住判断：
  - Telegram 发送失败
  - 路由 / 绑定问题
  - 长任务占住会话
  - 上下文 / token 过重
- 让诊断结果进入恢复事件和待通知队列
- 设计恢复后通知回对应窗口的原生 OpenClaw 流程
- 支持主窗口、群窗口、兜底窗口的不同通知策略

## 适用场景

适合以下需求：

- Telegram 聊天窗口经常卡住
- 主窗口和群窗口都在同一台 OpenClaw 主机上
- 想做 Telegram 自愈，而不只是手动重启 gateway
- 想在恢复后通知回正确的 DM / 群
- 想把“卡住”区分成网络、任务、上下文、路由四类
- 想把整套方案整理成可复制的生产 SOP

## 仓库结构

```text
telegram-selfheal-notification/
  SKILL.md
  references/
    architecture.md
    checklist.md
    templates.md
```

## 打包

```bash
python3 /opt/homebrew/lib/node_modules/openclaw/skills/skill-creator/scripts/package_skill.py \
  telegram-selfheal-notification
```

## 发布方式

打包出来的 `.skill` 文件可直接附加到 GitHub Release。

---

This repository packages an AgentSkill for **Telegram self-healing, four-way stuck-case diagnosis, window attribution, and post-recovery notification routing** in OpenClaw.

It focuses on the real operational problem where:

- Telegram chats look stuck
- the gateway is alive but `sendMessage` / `sendChatAction` fails
- restarting alone is not enough
- you also need to know **which DM or group was affected**
- not every stuck case is transport failure — it may also be:
  - routing / binding problems
  - long-running tasks
  - context / token-heavy sessions
- after recovery, the notification should go back to the **correct window**

## Core capabilities

- detect Telegram delivery failures
- auto-restart the OpenClaw gateway
- record structured recovery events
- attribute the likely affected Telegram window
- classify stuck cases into four common causes
- carry diagnosis results into recovery events and pending notification records
- define a native OpenClaw post-recovery notification path
- support different strategies for main DM, groups, and fallback delivery

## Use cases

Use this skill when you need to:

- reduce Telegram chat freeze incidents
- protect both main DM and Telegram groups on one OpenClaw host
- build self-healing instead of manual restart-only operations
- route restore notifications back to the right DM or group
- distinguish transport failures from routing issues, busy tasks, and heavy-context sessions
- package the workflow as a repeatable production SOP

## Repository layout

```text
telegram-selfheal-notification/
  SKILL.md
  references/
    architecture.md
    checklist.md
    templates.md
```

## Packaging

```bash
python3 /opt/homebrew/lib/node_modules/openclaw/skills/skill-creator/scripts/package_skill.py \
  telegram-selfheal-notification
```

## License

MIT
