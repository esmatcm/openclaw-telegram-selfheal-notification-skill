# Post-install local setup

Installing the skill package is not enough by itself.

This skill documents and packages the workflow, but the actual self-heal behavior still depends on **local machine setup** on the target OpenClaw host.

## Required local setup after skill installation

After installing the skill, the operator must still configure the local environment on the machine that runs OpenClaw.

Typical items include:

- place the self-heal scripts under the local OpenClaw runtime/bin area
- configure cron or another scheduler to run the scripts periodically
- create or update the local window map JSON
- ensure the target agent workspaces contain the needed local prompt files
- confirm the local gateway service is installed and running
- verify the bot/account used by that host can actually send to the target DM/group

## Typical local files

Examples:

- `~/.openclaw/bin/telegram-selfheal.sh`
- `~/.openclaw/bin/telegram-stuck-diagnose.sh`
- `~/.openclaw/bin/telegram-selfheal-attribution.sh`
- `~/.openclaw/bin/telegram-selfheal-dispatch.sh`
- `~/.openclaw/bin/telegram-session-heal.sh`
- `~/.openclaw/runtime/config/telegram-window-map.json`
- local `AGENTS.md` / `SOUL.md` / `USER.md` in the target agent workspace

## Important reminder

If someone only installs this skill but does not perform the local setup on the actual OpenClaw host, the self-heal system will **not** become active automatically.

In short:

- skill install = documentation / packaged workflow available
- local setup = system actually running

Both are required.
