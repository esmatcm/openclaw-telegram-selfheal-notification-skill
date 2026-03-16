# OpenClaw Telegram 自愈通知 Skill / OpenClaw Telegram Self-Heal Notification Skill

把 **OpenClaw Telegram 发送失败自愈、4 类卡住判断、session 软自愈、窗口归因、恢复后通知** 整理成一份可复用、可发布的 AgentSkill。

这份仓库主要解决的是：

- Telegram 私聊或群窗口“像卡住”
- gateway 还活着，但 `sendMessage` / `sendChatAction` 失败
- 单纯重启 gateway 还不够，还需要知道**影响的是哪个窗口**
- 不是所有卡住都是 Telegram 故障，还可能是：
  - 长任务占住
  - 上下文 / token 过重
  - 路由 / 绑定问题
- 过重且陈旧的 session 需要被识别并建议 `/new`
- session-heal 阈值需要足够灵敏，不能只抓 stale+heavy
- 恢复后希望把通知回到**对应主窗口 / 对应群**
- **安装 skill 后，还必须在目标 OpenClaw 主机的本地环境补配置，系统才会真正启动**

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
- 增加 **session 软自愈**：
  - `heavy_fast_path`：很重的 session 更早触发 `/new`
  - `stale_heavy_path`：较久未更新且偏重的 session 触发 `/new`
- 让诊断结果进入恢复事件和待通知队列
- 设计恢复后通知回对应窗口的原生 OpenClaw 流程
- 支持主窗口、群窗口、兜底窗口的不同通知策略
- 包含 live 验证要点：闭环成功与 `chat not found` 类失败的判读

## 重要说明：安装 skill 不等于系统已启动

安装这份 skill 之后，**还必须**到实际运行 OpenClaw 的本地主机继续完成配置，例如：

- 放置本地脚本到 `~/.openclaw/bin/`
- 配置 cron / scheduler
- 建立 `telegram-window-map.json`
- 补本地 agent workspace 的 `AGENTS.md / SOUL.md / USER.md`
- 确认本机 gateway 正常运行
- 确认该主机上的 bot / 账号确实能发到对应窗口

也就是说：

- **skill 安装** = 拿到这套方法与打包内容
- **本地环境配置完成** = 自愈系统真正开始工作

## 仓库结构

```text
telegram-selfheal-notification/
  SKILL.md
  references/
    architecture.md
    checklist.md
    templates.md
    post-install.md
```

## 打包

```bash
python3 /opt/homebrew/lib/node_modules/openclaw/skills/skill-creator/scripts/package_skill.py \
  telegram-selfheal-notification
```

## 发布方式

打包出来的 `.skill` 文件可直接附加到 GitHub Release。

---

This repository packages an AgentSkill for **Telegram self-healing, four-way stuck-case diagnosis, soft session healing, window attribution, and post-recovery notification routing** in OpenClaw.

It focuses on the real operational problem where:

- Telegram chats look stuck
- the gateway is alive but `sendMessage` / `sendChatAction` fails
- restarting alone is not enough
- you also need to know **which DM or group was affected**
- not every stuck case is transport failure — it may also be:
  - routing / binding problems
  - long-running tasks
  - context / token-heavy sessions
- stale-heavy or very-heavy sessions should trigger a soft-heal recommendation such as `/new`
- after recovery, the notification should go back to the **correct window**
- **installing the skill alone is not enough; the real OpenClaw host still needs local setup**

## Core capabilities

- detect Telegram delivery failures
- auto-restart the OpenClaw gateway
- record structured recovery events
- attribute the likely affected Telegram window
- classify stuck cases into four common causes
- add soft session-heal recommendations for stale-heavy and very-heavy sessions
- carry diagnosis results into recovery events and pending notification records
- define a native OpenClaw post-recovery notification path
- support different strategies for main DM, groups, and fallback delivery
- include live-validation thinking for success and `chat not found` failure cases

## Important note: skill install is not enough

After installing this skill, the operator still needs to finish local setup on the actual OpenClaw host, such as:

- place runtime scripts in the local `~/.openclaw/bin/` area
- configure cron or another scheduler
- create the local window map JSON
- add local prompt files in the target agent workspace
- confirm the local gateway is installed and running
- verify that the bot/account on that host can actually send to the intended DM/group

In short:

- **skill install** = packaged workflow available
- **local setup complete** = system actually active

## Repository layout

```text
telegram-selfheal-notification/
  SKILL.md
  references/
    architecture.md
    checklist.md
    templates.md
    post-install.md
```

## Packaging

```bash
python3 /opt/homebrew/lib/node_modules/openclaw/skills/skill-creator/scripts/package_skill.py \
  telegram-selfheal-notification
```

## License

MIT
