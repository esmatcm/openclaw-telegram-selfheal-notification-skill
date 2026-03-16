# Templates

## Event JSONL example

```json
{
  "id": "20260316-173001-001",
  "time": "2026-03-16T17:30:01+08:00",
  "kind": "gateway_restart_success",
  "apiReachable": false,
  "recentFail": true,
  "gatewayRestarted": true,
  "symptoms": "telegram sendMessage failed;telegram final reply failed",
  "targetGuess": "telegram:group:-5188818850",
  "diagnosisReason": "telegram_transport_failure",
  "diagnosisConfidence": "high",
  "notified": false
}
```

## Session-heal event example

```json
{
  "id": "SESSION-HEAL-qun3-1773663253",
  "time": "2026-03-16T21:02:26+08:00",
  "kind": "session_heal_recommended",
  "apiReachable": true,
  "recentFail": false,
  "gatewayRestarted": false,
  "symptoms": "stale_heavy_session idle=2893.6 size=698657",
  "targetGuess": "telegram:group:-5188818850",
  "diagnosisReason": "context_or_token_heavy_session",
  "diagnosisConfidence": "medium",
  "sessionHealSuggested": true,
  "recommendation": "/new",
  "notified": false
}
```

## Pending notification example

```json
{
  "time": "2026-03-16T17:31:45+08:00",
  "target": "telegram:group:-5188818850",
  "message": "检测到 Telegram 异常，已自动恢复。诊断：更像长任务占住会话（置信度：medium）。",
  "sourceEventId": "20260316-173001-001",
  "diagnosisReason": "long_running_task_busy",
  "diagnosisConfidence": "medium"
}
```

## Session-heal pending example

```json
{
  "time": "2026-03-16T21:02:26+08:00",
  "target": "telegram:group:-5188818850",
  "message": "检测到会话疑似卡重：更像上下文 / token 过重（置信度：medium）。建议在对应窗口执行 /new 进行软自愈。",
  "sourceEventId": "SESSION-HEAL-qun3-1773663253",
  "diagnosisReason": "context_or_token_heavy_session",
  "diagnosisConfidence": "medium"
}
```

## Main DM message

```text
检测到 Telegram 异常，已自动恢复。诊断：更像 Telegram 发送失败（置信度：high）。
```

## Group message

```text
检测到 Telegram 异常，已自动恢复。诊断：更像长任务占住会话（置信度：medium）。
```

## Session-heal message

```text
检测到会话疑似卡重：更像上下文 / token 过重（置信度：medium）。建议在对应窗口执行 /new 进行软自愈。
```

## Fallback message

```text
检测到 Telegram 异常，已自动恢复。诊断：原因暂时不明确（置信度：low）。
```

## Live validation failure example

```text
[telegram] message failed: chat not found
```

Interpretation:
The dispatch path executed correctly, but the target chat is invalid or unavailable for the current bot/runtime.
