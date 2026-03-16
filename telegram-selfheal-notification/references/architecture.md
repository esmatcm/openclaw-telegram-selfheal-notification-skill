# Architecture

## Purpose

This skill documents a layered design for Telegram self-healing in OpenClaw.

## Layers

1. Detection
2. Self-heal restart logic
3. Structured event recording
4. Window attribution
5. Native OpenClaw notification

## Why layered design matters

A shell script can reliably detect and restart, but it should not be trusted to route user-facing Telegram notifications back into the correct OpenClaw conversation surface.

The safer split is:

- shell/system: operational recovery
- OpenClaw: conversation-aware messaging

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
