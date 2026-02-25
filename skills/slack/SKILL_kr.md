---
name: slack
description: OpenClaw에서 Slack 도구를 제어하여 메시지에 반응을 남기거나, 채널/DM에서 항목을 고정/해제하고 싶을 때 사용합니다.
metadata: { "openclaw": { "emoji": "💬", "requires": { "config": ["channels.slack"] } } }
---

# Slack 액션

## 개요
`slack` 도구를 사용하여 메시지 반응(리액션), 고정 메시지 관리, 메시지 전송/수정/삭제, 멤버 정보 조회를 수행합니다. 이 도구는 OpenClaw에 설정된 봇 토큰을 사용합니다.

## 주요 입력 항목
- `channelId`: Slack 채널 ID.
- `messageId`: Slack 메시지 타임스탬프 (예: `1712023032.1234`).
- `emoji`: 반응을 남길 이모지 (유니코드 또는 `:이름:` 형식).
- `to`: 전송 대상 (`channel:<id>` 또는 `user:<id>`).
- `content`: 메시지 내용.

## 주요 액션 예시

### 메시지에 반응 남기기
```json
{
  "action": "react",
  "channelId": "C123",
  "messageId": "1712023032.1234",
  "emoji": "✅"
}
```

### 메시지 전송하기
```json
{
  "action": "sendMessage",
  "to": "channel:C123",
  "content": "OpenClaw에서 보낸 메시지입니다."
}
```

### 메시지 고정하기 (Pin)
```json
{
  "action": "pinMessage",
  "channelId": "C123",
  "messageId": "1712023032.1234"
}
```

---
상세한 액션 목록이나 추가 명령어는 영문 문서를 참조하세요.
---
