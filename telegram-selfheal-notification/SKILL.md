---
name: telegram-selfheal-notification
description: Design and implement OpenClaw Telegram self-healing with gateway restart protection, four-way stuck-case diagnosis, event logging, window attribution, restore-notification routing, and local post-install setup requirements. Use when Telegram send failures cause stuck chats, when you need auto-recovery for gateway sendMessage/sendChatAction failures, or when you want per-window recovery notifications instead of one shared fallback alert.
---

# Telegram Self-Heal Notification

Build a Telegram recovery workflow for OpenClaw that does more than just restart the gateway.

## Goal

When a Telegram chat appears stuck, the system should:

1. classify the likely cause
2. auto-restart the gateway safely when transport failure is the problem
3. record structured recovery events
4. attribute the incident to the likely affected window
5. notify the correct Telegram DM or group after recovery
6. remind the operator that local host setup is required after installing the skill

## Important post-install rule

Installing this skill package alone is not enough.

After installation, the operator must still configure the **local OpenClaw environment on the actual host** where the bot/runtime runs.

If the local host setup is not completed, the self-heal system will not become active even if the skill is installed successfully.

Read `references/post-install.md` before considering the setup finished.

## Four-way diagnosis

Before acting, distinguish among these common causes:

1. Telegram transport failure
2. routing or binding issue
3. long-running task busy
4. context or token-heavy session

Do not treat all stuck windows as Telegram API outages.

## Core principle

Do not rely on a single shell script to both recover and notify.

Split responsibilities into layers:

- shell/system layer → detection + restart + event queue + diagnosis
- OpenClaw layer → message routing back to the correct window

## Recommended architecture

### Layer 1: detection

Watch for Telegram transport problems such as:

- `telegram sendMessage failed`
- `telegram final reply failed`
- `telegram slash final reply failed`
- `telegram sendChatAction failed`
- `UND_ERR_CONNECT_TIMEOUT`

Also test reachability to `https://api.telegram.org`.

### Layer 2: diagnosis

Classify the stuck case before deciding what to do.

Suggested labels:

- `telegram_transport_failure`
- `routing_or_binding_issue`
- `long_running_task_busy`
- `context_or_token_heavy_session`
- `unknown_or_transient`

Use best-effort heuristics from:

- recent Telegram failure logs
- `not-allowed` or binding-related log lines
- recent session file activity
- recent session file size as a rough heavy-context signal

### Layer 3: self-heal

Use a cooldown-based restart strategy.

Recommended defaults:

- check every 60 seconds
- restart after 2 consecutive failed checks
- use a 5-minute cooldown to avoid restart loops

Only restart the gateway when the evidence points to transport failure.

### Layer 4: structured event logging

Write recovery events as JSONL, not just plain text.

Suggested fields:

- `id`
- `time`
- `kind`
- `apiReachable`
- `recentFail`
- `gatewayRestarted`
- `symptoms`
- `targetGuess`
- `diagnosisReason`
- `diagnosisConfidence`
- `notified`

### Layer 5: attribution

Try to infer which window was affected.

Typical targets:

- `telegram:dm:<userId>`
- `telegram:group:<chatId>`

Use the best available evidence:

1. recent `chatId` or `conversationId` in logs
2. nearby group title mentions
3. agent bindings for known Telegram groups
4. fallback to the main DM if attribution is uncertain

### Layer 6: native OpenClaw notification

Do not notify via raw shell/curl.

Use OpenClaw's own messaging layer to deliver the restore message after recovery.

If attribution is exact, send to the matching window.
If attribution is uncertain, send to the main DM.
Do not guess and post to the wrong group.

Carry diagnosis into the pending notification text so the operator sees not only that the system recovered, but also what kind of stuck case it most likely was.

### Layer 7: soft session-heal

Detect stale-heavy or very-heavy sessions and generate recommendation events.

Suggested logic:

- `heavy_fast_path`: high size + short idle
- `stale_heavy_path`: medium size + longer idle

For example:

- `size > 1.5MB` and `idle > 180s`
- `size > 600KB` and `idle > 900s`

The recommendation should usually be `/new`.

## Recommended files

Examples:

- self-heal script
- diagnosis script
- attribution script
- dispatch script
- session-heal script
- window map JSON
- event JSONL log
- pending notification queue
- local prompt files in the target agent workspace

Suggested paths:

- `~/.openclaw/bin/telegram-selfheal.sh`
- `~/.openclaw/bin/telegram-stuck-diagnose.sh`
- `~/.openclaw/bin/telegram-selfheal-attribution.sh`
- `~/.openclaw/bin/telegram-selfheal-dispatch.sh`
- `~/.openclaw/bin/telegram-session-heal.sh`
- `~/.openclaw/runtime/config/telegram-window-map.json`
- `~/.openclaw/runtime/logs/telegram-selfheal-events.jsonl`
- `~/.openclaw/runtime/logs/telegram-selfheal-pending.jsonl`
- `~/.openclaw/runtime/logs/telegram-stuck-diagnosis.json`
- local `AGENTS.md`, `SOUL.md`, `USER.md` in the active agent workspace

## Notification rules

Use these principles:

- notify only after recovery, not during transport failure
- deduplicate by source event id
- aggregate repeated failures within a short window
- if target attribution is weak, notify the main DM instead of a group
- include diagnosis and confidence in the pending notification payload

## Example restore messages

### Main DM

```text
检测到 Telegram 异常，已自动恢复。诊断：更像 Telegram 发送失败（置信度：high）。
```

### Group window

```text
检测到 Telegram 异常，已自动恢复。诊断：更像长任务占住会话（置信度：medium）。
```

### Session-heal

```text
检测到会话疑似卡重：更像上下文 / token 过重（置信度：medium）。建议在对应窗口执行 /new 进行软自愈。
```

### Fallback

```text
检测到 Telegram 异常，已自动恢复。诊断：原因暂时不明确（置信度：low）。
```

## Use this skill when

- Telegram chats appear stuck but gateway is still running
- send failures are intermittent and require self-heal logic
- one OpenClaw host serves both DM and group Telegram windows
- you want recovery notices routed back to the matching window
- you want a production-grade design instead of a restart-only shell script
- you need to distinguish transport failures from routing issues, busy tasks, and heavy-context sessions
- you also want the skill to remind installers about the required local host configuration after install

## References

Read these before implementation:

- `references/architecture.md`
- `references/checklist.md`
- `references/templates.md`
- `references/post-install.md`
