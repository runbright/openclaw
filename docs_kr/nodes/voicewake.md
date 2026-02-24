---
summary: "글로벌 음성 웨이크 워드 (Gateway 소유) 및 노드 간 동기화 방법"
read_when:
  - 음성 웨이크 워드 동작 또는 기본값 변경 시
  - 웨이크 워드 동기화가 필요한 새 노드 플랫폼 추가 시
title: "음성 웨이크"
---

# 음성 웨이크 (글로벌 웨이크 워드)

OpenClaw는 **웨이크 워드를 Gateway가 소유하는 단일 글로벌 목록**으로 취급합니다.

- **노드별 커스텀 웨이크 워드는 없습니다**.
- **모든 노드/앱 UI가 목록을 편집할 수 있으며**; 변경사항은 Gateway에 의해 저장되고 모든 곳에 브로드캐스트됩니다.
- 각 기기는 여전히 자체 **Voice Wake 활성화/비활성화** 토글을 유지합니다 (로컬 UX + 권한이 다름).

## 저장소 (Gateway 호스트)

웨이크 워드는 gateway 머신에 다음 위치에 저장됩니다:

- `~/.openclaw/settings/voicewake.json`

형태:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## 프로토콜

### 메서드

- `voicewake.get` -> `{ triggers: string[] }`
- `voicewake.set` 매개변수 `{ triggers: string[] }` -> `{ triggers: string[] }`

참고:

- 트리거는 정규화됩니다 (공백 제거, 빈 항목 삭제). 빈 목록은 기본값으로 폴백됩니다.
- 안전을 위해 제한이 적용됩니다 (수/길이 상한).

### 이벤트

- `voicewake.changed` 페이로드 `{ triggers: string[] }`

수신자:

- 모든 WebSocket 클라이언트 (macOS 앱, WebChat 등.)
- 모든 연결된 노드 (iOS/Android), 그리고 노드 연결 시 초기 "현재 상태" 푸시로도.

## 클라이언트 동작

### macOS 앱

- 글로벌 목록을 사용하여 `VoiceWakeRuntime` 트리거를 게이트합니다.
- Voice Wake 설정에서 "Trigger words"를 편집하면 `voicewake.set`을 호출하고, 다른 클라이언트와의 동기화를 위해 브로드캐스트에 의존합니다.

### iOS 노드

- `VoiceWakeManager` 트리거 감지를 위해 글로벌 목록을 사용합니다.
- 설정에서 Wake Words를 편집하면 (Gateway WS를 통해) `voicewake.set`을 호출하고 로컬 웨이크 워드 감지도 반응적으로 유지합니다.

### Android 노드

- 설정에서 Wake Words 편집기를 노출합니다.
- 편집 내용이 모든 곳에 동기화되도록 Gateway WS를 통해 `voicewake.set`을 호출합니다.
