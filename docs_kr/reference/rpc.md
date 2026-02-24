---
summary: "외부 CLI (signal-cli, 레거시 imsg) 및 게이트웨이 패턴을 위한 RPC 어댑터"
read_when:
  - Adding or changing external CLI integrations
  - Debugging RPC adapters (signal-cli, imsg)
title: "RPC 어댑터"
---

# RPC 어댑터

OpenClaw은 JSON-RPC를 통해 외부 CLI를 통합합니다. 현재 두 가지 패턴이 사용됩니다.

## 패턴 A: HTTP 데몬 (signal-cli)

- `signal-cli`는 HTTP를 통한 JSON-RPC로 데몬으로 실행됩니다.
- 이벤트 스트림은 SSE (`/api/v1/events`).
- 상태 프로브: `/api/v1/check`.
- `channels.signal.autoStart=true`일 때 OpenClaw이 생명주기를 관리합니다.

설정 및 엔드포인트는 [Signal](/channels/signal)을 참조하세요.

## 패턴 B: stdio 자식 프로세스 (레거시: imsg)

> **참고:** 새로운 iMessage 설정에는 [BlueBubbles](/channels/bluebubbles)를 사용하세요.

- OpenClaw은 `imsg rpc`를 자식 프로세스로 생성합니다 (레거시 iMessage 통합).
- JSON-RPC는 stdin/stdout을 통해 라인 구분됩니다 (라인당 하나의 JSON 객체).
- TCP 포트 불필요, 데몬 불필요.

사용되는 핵심 메서드:

- `watch.subscribe` → 알림 (`method: "message"`)
- `watch.unsubscribe`
- `send`
- `chats.list` (프로브/진단)

레거시 설정 및 주소 지정은 [iMessage](/channels/imessage)를 참조하세요 (`chat_id` 권장).

## 어댑터 가이드라인

- 게이트웨이가 프로세스를 소유합니다 (시작/중지가 프로바이더 생명주기에 연결).
- RPC 클라이언트를 탄력적으로 유지: 타임아웃, 종료 시 재시작.
- 표시 문자열보다 안정적인 ID (예: `chat_id`)를 선호.
