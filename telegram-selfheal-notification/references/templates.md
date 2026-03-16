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
  "notified": false
}
```

## Pending notification example

```json
{
  "time": "2026-03-16T17:31:45+08:00",
  "target": "telegram:group:-5188818850",
  "message": "检测到该窗口刚刚出现 Telegram 发送异常，已自动恢复，目前已恢复可用。",
  "sourceEventId": "20260316-173001-001"
}
```

## Main DM message

```text
检测到 Telegram 发送异常，已自动恢复，目前主窗口已恢复可用。
```

## Group message

```text
检测到该窗口刚刚出现 Telegram 发送异常，已自动恢复，目前已恢复可用。
```

## Fallback message

```text
检测到 Telegram 发送异常，已自动恢复，但本次无法精确判断受影响窗口，请留意最近活跃窗口。
```
