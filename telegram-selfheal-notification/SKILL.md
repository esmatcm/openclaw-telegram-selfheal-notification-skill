---
name: telegram-selfheal-notification
description: Design and implement OpenClaw Telegram self-healing with gateway restart protection, event logging, window attribution, and restore-notification routing back to the correct DM or Telegram group. Use when Telegram send failures cause stuck chats, when you need auto-recovery for gateway sendMessage/sendChatAction failures, or when you want per-window recovery notifications instead of one shared fallback alert.
---

# Telegram Self-Heal Notification

Build a Telegram recovery workflow for OpenClaw that does more than just restart the gateway.

## Goal

When Telegram delivery fails, the system should:

1. detect the failure
2. auto-restart the gateway safely
3. record structured recovery events
4. attribute the incident to the likely affected window
5. notify the correct Telegram DM or group after recovery

## Core principle

Do not rely on a single shell script to both recover and notify.

Split responsibilities into layers:

- shell/system layer → detection + restart + event queue
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

### Layer 2: self-heal

Use a cooldown-based restart strategy.

Recommended defaults:

- check every 60 seconds
- restart after 2 consecutive failed checks
- use a 5-minute cooldown to avoid restart loops

### Layer 3: structured event logging

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
- `notified`

### Layer 4: attribution

Try to infer which window was affected.

Typical targets:

- `telegram:dm:<userId>`
- `telegram:group:<chatId>`

Use the best available evidence:

1. recent `chatId` or `conversationId` in logs
2. nearby group title mentions
3. agent bindings for known Telegram groups
4. fallback to the main DM if attribution is uncertain

### Layer 5: native OpenClaw notification

Do not notify via raw shell/curl.

Use OpenClaw's own messaging layer to deliver the restore message after recovery.

If attribution is exact, send to the matching window.
If attribution is uncertain, send to the main DM.
Do not guess and post to the wrong group.

## Recommended files

Examples:

- self-heal script
- window map JSON
- event JSONL log
- pending notification queue

Suggested paths:

- `~/.openclaw/bin/telegram-selfheal.sh`
- `~/.openclaw/bin/telegram-selfheal-attribution.sh`
- `~/.openclaw/bin/telegram-selfheal-dispatch.sh`
- `~/.openclaw/runtime/config/telegram-window-map.json`
- `~/.openclaw/runtime/logs/telegram-selfheal-events.jsonl`
- `~/.openclaw/runtime/logs/telegram-selfheal-pending.jsonl`

## Notification rules

Use these principles:

- notify only after recovery, not during transport failure
- deduplicate by source event id
- aggregate repeated failures within a short window
- if target attribution is weak, notify the main DM instead of a group

## Example restore messages

### Main DM

```text
检测到 Telegram 发送异常，已自动恢复，目前主窗口已恢复可用。
```

### Group window

```text
检测到该窗口刚刚出现 Telegram 发送异常，已自动恢复，目前已恢复可用。
```

### Fallback

```text
检测到 Telegram 发送异常，已自动恢复，但本次无法精确判断受影响窗口，请留意最近活跃窗口。
```

## Use this skill when

- Telegram chats appear stuck but gateway is still running
- send failures are intermittent and require self-heal logic
- one OpenClaw host serves both DM and group Telegram windows
- you want recovery notices routed back to the matching window
- you want a production-grade design instead of a restart-only shell script

## References

Read these before implementation:

- `references/architecture.md`
- `references/checklist.md`
- `references/templates.md`
