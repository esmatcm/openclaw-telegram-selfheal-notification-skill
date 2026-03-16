# Architecture

## Purpose

This skill documents a layered design for Telegram self-healing in OpenClaw.

## Layers

1. Detection
2. Diagnosis
3. Gateway self-heal restart logic
4. Structured event recording
5. Window attribution
6. Native OpenClaw notification
7. Soft session-heal recommendation

## Why layered design matters

A shell script can reliably detect and restart, but it should not be trusted to route user-facing Telegram notifications back into the correct OpenClaw conversation surface by itself.

The safer split is:

- shell/system: operational recovery, diagnosis, event generation
- OpenClaw: conversation-aware messaging

## Diagnosis model

Before acting, distinguish among:

- `telegram_transport_failure`
- `routing_or_binding_issue`
- `long_running_task_busy`
- `context_or_token_heavy_session`
- `unknown_or_transient`

## Attribution model

Preferred target keys:

- `telegram:dm:<userId>`
- `telegram:group:<chatId>`

Fallback order:

1. exact chat id in recent logs
2. known group title in recent logs
3. known binding map
4. main DM fallback

## Delivery model

Only send restore notifications after Telegram delivery is healthy again.

Do not emit alerts during the outage itself if Telegram transport is the failed dependency.

## Soft session-heal model

If a session is both stale and heavy, create a recommendation event instead of restarting the gateway.

Recommended behavior:

- detect stale-heavy sessions by idle time + file size thresholds
- write a `session_heal_recommended` event
- set recommendation to `/new`
- route the recommendation back to the affected window after normal notification handling

## Live validation notes

A correct end-to-end test should confirm:

- event creation
- pending generation
- native OpenClaw send path
- event/pending write-back after success or failure

A `chat not found` result means the notification path executed, but the target chat is not valid for that bot/runtime.
