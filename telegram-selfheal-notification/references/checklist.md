# Checklist

Use this checklist before declaring the setup complete.

- Gateway self-heal script exists and is executable.
- Check interval and cooldown are configured.
- Telegram API reachability is tested.
- Log-based Telegram failure patterns are monitored.
- Structured JSONL event file is written.
- Window map file exists and matches current DM/group targets.
- Attribution logic fills `targetGuess` on new events.
- Pending notification queue is populated only after recovery-worthy events.
- Native OpenClaw notification path is defined.
- Main DM fallback exists for low-confidence attribution.
- Duplicate event suppression is implemented.
- Repeated failures are aggregated instead of spamming groups.
